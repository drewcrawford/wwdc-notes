#metal 

Simulate rays of light bouncing around a scene
Used in games, offline rendering, and more
Performance is critical

Shader function such as ocmpute or fragment.

1.  generate rays
2. intersector
3. acceleration structure
4. intersection result
5. produces image

[[Discover ray tracing with metal]]
[[Enhance your app with Metal ray tracing]]

# Ray tracing performance
* per-primitive data
* buffers from intersection function tables
* indirect command buffers

## per primitive data
store data directly in the acceleration structure
reduced memory indirections
simplified data structures

alpha testing with intersection functions
ultimate goal is to sampel from the texture associated with the triangle and test if allwoed to continue.
```cpp
float alpha = texture.sample(sampler, UV).w;
return alpha >= 0.5f;
```

typically, you need to acces a number of intermediate buffers to get this information.
1.  Fiurst, texture is stored in some material structure.  Several materials are packed into a buffer, it is impractical to store a material per-primitive.
2. instead, material ID buffer.
3. For UBs, you need to load them from a buffer and interpolate them.
4. Each instance may have its own materials and UV mappings.  So instance points into material ID and UV buffer
Fairly complex buffer setup with many layers of indirection.  Leads to cache misses that affect performance.

original implementation
```cpp
[[intersection(triangle, raytracing::triangle_data, raytracing::instancing)]]
bool alphaTestIntersection(float2               coordinates    [[barycentric_coord]],
                           unsigned int         primitiveIndex [[primitive_id]],
                           unsigned int         instanceIndex  [[instance_id]],
                           device GlobalData   *globalData     [[buffer(1)]],
                           device InstanceData *instanceData   [[buffer(0)]])
{
    device Material *materials = globalData->materials;
    InstanceData instance = instanceData[instanceIndex];
    float2 UV = calculateSamplingCoords(coordinates,
                                        instance.uvs[primitiveIndex * 3 + 0],
                                        instance.uvs[primitiveIndex * 3 + 1],
                                        instance.uvs[primitiveIndex * 3 + 2]);
    int materialIndex = instance.materialIndices[primitiveIndex];
    float alpha = materials[materialIndex].texture.sample(sam, UV).w;
    return alpha >= 0.5f;
}
```

How to simplify this code and improve its performance using per-primitive data.  You can simply store only the data the intersection function will need.
```cpp
struct PrimitiveData
{
    texture2d<float> texture;
    float2 uvs[3];
};
```

you provide this when building the acceleration structure and the intersection function receives a pointer to this data.

replace all buffer arguments with the primitive data pointer

```cpp
// Alpha testing intersection function
[[intersection(triangle, raytracing::triangle_data, raytracing::instancing)]]
bool alphaTestIntersection(float2               coordinates    [[barycentric_coord]],
                     const device PrimitiveData *primitiveData [[primitive_data]])
{
    PrimitiveData ppd = *primitiveData;
    float2 UV = calculateSamplingCoords(coordinates,
                                        ppd.uvs[0],
                                        ppd.uvs[1],
                                        ppd.uvs[2]);
    float alpha = ppd.texture.sample(sam, UV).w;
    return alpha >= 0.5f;
}
```

One load from per primitive data pointer.  Only memory access neeeded.

Simply use texture directly withotu paying the cost of additional memory indirection.

```swift
geometryDescriptor.primitiveDataBuffer = primitiveDataBuffer
geometryDescriptor.primitiveDataElementSize = MemoryLayout<PrimitiveData>.size
geometryDescriptor.primitiveDataStride = MemoryLayout<PrimitiveData>.stride
geometryDescriptor.primitiveDataBufferOffset = primitiveDataOffset
```

```cpp
// Intersection function argument:
const device void *primitiveData [[primitive_data]]

// Intersection result:
primitiveData = intersection.primitive_data;

// Intersection query:
primitiveData = query.get_candidate_primitive_data();
primitiveData = query.get_committed_primitive_data();
```

For shading in addition to intersection testing.
"shading of intersection results"

10-16% performance improvement.

## Buffers from intersection function tables
Apps share bindings between intersection functions and ray tracing kernel.
```cpp
device int *buffer = intersectionFunctionTable.get_buffer<device int *>(index);

visible_function_table<uint(uint)> table =
    intersectionFunctionTable.get_visible_function_table<uint(uint)>(index);

uint result = table[0](parameter);
```

## Indirect command buffers
[[Modern rendering with Metal]]

```swift
let icbDescriptor = MTLIndirectCommandBufferDescriptor()

icbDescriptor.supportRayTracing = true
```
```cpp
for (index, accelerationStructure) in accelerationStructures.enumerated() {
    encoder.build(accelerationStructure: accelerationStructure,
                  descriptor: descriptors[index],
                  scratchBuffer: scratchBuffers[index % numScratchBuffers],
                  scratchBufferOffset: 0)
}
```

Can now use ray tracing from there as usual.

# Acceleration structures
datastructures which accelerate intersection process.  Recursively partitioning space os we can quickly find which triangles are likely.

2 types:  Primitive and instance.
* plane
* cube
* sphere
* triangle mesh
instance acceleration structure => copies of primitive acceleration structure.
Use array of transformation structures + primitive structures to build an instance structure.

## Building
need to build primitive acceleration structures for all models.  Build as many of your primitives as possible at load time to save time.  Use an instance structure to add/remove from scene as needed.

Will need ot update some accel structures.  Refit or rebuild.  Refit where possible.  Do a rebuild since objects may have been added or removed.  Full rebuild is fine these days since theres' only one instance acceleration structure.

This year we've improved performance for all cases.  2.3x faster builds on apple silicon.  Refitting is 38% faster on apple silicon.

Some apps build lots of small acceleration structures.  Not enough work to fill the GPU.  Builds now automatically performed in parllel on apple silicon.  2.8x faster parallel builds in parallel.  Compacting, refitting, etc.

Guidelines.  Use a single command encoder for many builds.  Ensure that you are lookign through a small pool of scratch buffers and not using the same one per build.

* vertex formats
* transformation amtrices
* heaps

## Vertex formats
Reduced precision vertex formats
Reduced memory usage.  Now we support wide range of formats.  Half floating point, 2-component formats for planar geometry, etc.

With new vertex formats, acceleration structure builds can now be in vertex data in any supported format.






## Transformation matrices
* transform vertex data before building structure

Mesh, calculate its bounds, and then scale them to a 0-1 range.  Then use one of the normlized formats to store the mesh, reducing the amount of space it takes up on disk.

At runtime, you provide the matrix that builds scale and offset to un-squash.

```swift
let geometryDescriptor = MTLAccelerationStructureTriangleGeometryDescriptor()

geometryDescriptor.vertexFormat = .uint1010102Normalized
```
```swift
var scaleTransform =
    MTLPackedFloat4x3(columns: (
        MTLPackedFloat3Make( scale.x,      0.0,      0.0),
        MTLPackedFloat3Make(     0.0,  scale.y,      0.0),
        MTLPackedFloat3Make(     0.0,      0.0,  scale.z),
        MTLPackedFloat3Make(offset.x, offset.y, offset.z))

let transformBuffer = device.makeBuffer(length: MemoryLayout<MTLPackedFloat4x3>.size,
                                        options: .storageModeShared)!

transformBuffer.contents().copyMemory(from: &scaleTransform,
                                      byteCount: MemoryLayout<MTLPackedFloat4x3>.size)
```


```swift
let geometryDescriptor = MTLAccelerationStructureTriangleGeometryDescriptor()

geometryDescriptor.transformationMatrixBuffer = transformBuffer
geometryDescriptor.transformationMatrixBufferOffset = 0
```


Suppose boxes and spheres are both relatively simple meshes.  There's an overhead for each instance.  More often with overlapping instances.  To reduce the instance count, can build a single accel structure with both box and sphere.  Resulting primitive acceleration structure is a single instance and contains multiple types.  Better performing acceleration structure.

```swift
let sphereGeometryDescriptor = MTLAccelerationStructureTriangleGeometryDescriptor()
sphereGeometryDescriptor.vertexBuffer = sphereVertexBuffer
sphereGeometryDescriptor.indexBuffer = sphereIndexBuffer
sphereGeometryDescriptor.transformationMatrixBuffer = sphereTransformBuffer

let redBoxGeometryDescriptor = MTLAccelerationStructureTriangleGeometryDescriptor()
redBoxGeometryDescriptor.vertexBuffer = boxVertexBuffer
redBoxGeometryDescriptor.indexBuffer = boxIndexBuffer
redBoxGeometryDescriptor.transformationMatrixBuffer = redBoxTransformBuffer

let blueBoxGeometryDescriptor = MTLAccelerationStructureTriangleGeometryDescriptor()
blueBoxGeometryDescriptor.vertexBuffer = boxVertexBuffer
blueBoxGeometryDescriptor.indexBuffer = boxIndexBuffer blueBoxGeometryDescriptor.transformationMatrixBuffer = blueBoxTransformBuffer

let primitiveASDescriptor = MTLPrimitiveAccelerationStructureDescriptor()

primitiveASDescriptor.geometryDescriptors =
    [sphereGeometryDescriptor, redBoxGeometryDescriptor, blueBoxGeometryDescriptor]
```

## Heaps
Heap acceleration structure allocation
* more ocntrol over allocation
* reuse heap memory between allocations
* fewer calls to `useResource:`

Each time you want to use an instance acceleration structure, need to call `useResource:` for each primitive acceleration structure.  For alrge scenes this may be thousands of calls.

Call `useResources:` to reduce calls but you still need to maintain an array of structures.  Instead, you can allocate all structures from the same heap.  When you want to use the structure, call `useHeap` to reference all the acceleration structures.

```swift
let heap = device.makeHeap(descriptor: heapDescriptor)!

let accelerationStructure = heap.makeAccelerationStructure(descriptor: descriptor)

//or find out the size and alignment
let sizeAndAlign = device.heapAccelerationStructureSizeAndAlign(descriptor: descriptor)

let accelerationStructure = heap.makeAccelerationStructure(size: sizeAndAlign.size)
```

Remember to call `useHeap:` to make everything resident
Heaps default to untracked allocations
Synchronize using MTLFence across encoders, or MTLEvent across command buffers.

# Improvements to tool
* Acceleration structure viewer
* shader profiler
* shader debugger
* shader validation

## Acceleration structure viewer
xc 14 now supports debugging acceleration structures with primitive or instanced motion.
Can also visualize primitives.

demo

Primitives.  Useful to use per-primitive data API.  It's the button in the bottom right hand corner.  Select specific primitives for detailed inspection.  Find arrows to disclose data.  

## Shader profiler
per-pipeline execution timing costs.  On apple GPUs it provides omre granularity at the source level with execution costs per line.

intersection functions, visible functions, dynamic libraries.

## shader debugger
Can now follow execution all the way into visible function code.  Also for dynamic libraries.

## Shader validation
Diagnose runtime errors ont he GPU
* OOB access
* null texture reads
* invalid resource residency
* timeouts
* stakc overflow (new)
edit scheme => run => diagnostics => shader validation

calll stack is originally in device memory.  If function is not known at compile time, metal has to estimate the stack size.  e.g. ray tracing intersection function.  If using custom intersection functions, stack depth = 1.

If using function tables, also unknown at compile time.  dylib.  etc.  Function pointer.  To properly confiugre metal, must specify a call stack depth.  Improtant thing to remember is that when stack depth is too low, SO can happen which is UB.

If shader validation is enabled, such situations will be discovered.

* asv
* shader profiler
* shader debugger
* validation
[[Discover Metal debugging, profiling, and asset creation tools]]
[[Debug GPU-side errors in Metal]]
[[Metal Shader debugging and profiling]]

# Wrap up
ray tracing performance
acceleration structures
improvements to tools



