#metal 

# Ray tracing
Tracing the paths that rays take as they travle through a scene

# Process
1.  Generate rays
2.  Intersect with scene
3.  Shading (may generate additional rays to send back to 2)
4.  Image

See diagrmas in the slides

With new API, can combine all processes into a single compute kernel.  Therefore we don't need to read/write memory to pass intersections around.

Let's launch a 2d compute kernel with one thread per pixel.

```cpp
[[kernel]]
void rtKernel(primitive_acceleration_structure accelerationStructure [[buffer(0)]],
              /* ... */)
{
    // Generate ray
    ray r = generateCameraRay(tid);

    // Create an intersector
    intersector<triangle_data> intersector;

    // Intersect with scene
    intersection_result<triangle_data> intersection;

    intersection = intersector.intersect(r, accelerationStructure);

    // shading...
}
```

May need to divide up into more kernels to get good occupancy, please profile.

# Acceleration structure

Recursively partition space so we could eliminate triangles that cannot possibly intersect a given ray
Metal builds this for us

1.  Create descriptor
2.  Allocate storage
3.  Build

Primitive structures, instance structures.  We will do primitives for this example.  Individual geometry.  Each piece of geometry can ahve its own vertex buffer, index buffer, etc.

## Allocation stuff

```swift
let accelerationStructureDescriptor = MTLPrimitiveAccelerationStructureDescriptor()

// Create geometry descriptor(s)
let geometryDescriptor = MTLAccelerationStructureTriangleGeometryDescriptor()

geometryDescriptor.vertexBuffer = vertexBuffer
geometryDescriptor.triangleCount = triangleCount

accelerationStructureDescriptor.geometryDescriptors = [ geometryDescriptor ]
```

allocate a scratch buffer as well.  This is used during the accel init.

```swift
// Query for acceleration structure sizes
let sizes = device.accelerationStructureSizes(descriptor: accelerationStructureDescriptor)

// Allocate acceleration structure
let accelerationStructure =
    device.makeAccelerationStructure(size: sizes.accelerationStructureSize)!

// Allocate scratch buffer
let scratchBuffer = device.makeBuffer(length: sizes.buildScratchBufferSize,
                                      options: .storageModePrivate)!
```

`
## Building
```swift
// Create command buffer/encoder
let commandBuffer = commandQueue.makeCommandBuffer()!
let commandEncoder = commandBuffer.makeAccelerationStructureCommandEncoder()!

// Encode acceleration structure build
commandEncoder.build(accelerationStructure: accelerationStructure,
                     descriptor: accelerationStructureDescriptor,
                     scratchBuffer: scratchBuffer,
                     scratchBufferOffset: 0)

// Commit command buffer
commandEncoder.endEncoding()
commandBuffer.commit()
```



Builds now run entirely on the GPU timeline with no CPU synchronization.

```cpp
[[kernel]]
void rtKernel(primitive_acceleration_structure accelerationStructure [[buffer(0)]],
              /* ... */)
{
    // generate ray, create intersector...

    intersection = intersector.intersect(r, accelerationStructure);

    // shading...
}
```

```cpp
computeEncoder.setAccelerationStructure(accelerationStructure, bufferIndex: 0)
```

[[Ray tracing with metal - 19]]
[[Metal for ray tracing acceleration - 18]]


## Advanced topics
* Instancing - reduce memory usage
* Refitting - existing structures, to support dynamic geometry
* Compaction - reclaim significant amounts of memory once it's been built

## Demo

# Intersection functions
Customize how the intersector works?

Alpha testing

Note that when we cast a ray, we may hit a transparent pixel in the texture.  So rather than decide we've intersected with that model,
we have to continue casting a new ray until we hit an opaque texture.

Inside the intersector

1.  Traverse acceleration structure
2.  Update closest intersection
3.  Goto 1
4.  Eventually we return the closest intersection

Intersection functions basically allow you to filter the intersections we find.

```cpp
[[intersection(triangle, triangle_data)]]
bool alphaTestIntersectionFunction(uint primitiveIndex        [[primitive_id]],
                                   uint geometryIndex         [[geometry_id]],
                                   float2 barycentricCoords   [[barycentric_coord]],
                                   device Material *materials [[buffer(0)]])
{
    texture2d<float> alphaTexture = materials[geometryIndex].alphaTexture;

    float2 UV = interpolateUVs(materials[geometryIndex].UVs,
        primitiveIndex, barycentricCoords);

    float alpha = alphaTexture.sample(sampler, UV).x;

    return alpha > 0.5f;
}
```

Also can write bounding-box intersection functions
Invoked when ray intersects a user-provided bounding box
models custom primitives.  This is evidently the 'coarse' version of the intersection.

evidently they have a bezier curve bounding box intersection function demo

```swift
// Create a primitive acceleration structure descriptor
let accelerationStructureDescriptor = MTLPrimitiveAccelerationStructureDescriptor()

// Create one or more bounding box geometry descriptors:
let geometryDescriptor = MTLAccelerationStructureBoundingBoxGeometryDescriptor()

geometryDescriptor.boundingBoxBuffer = boundingBoxBuffer
geometryDescriptor.boundingBoxCount = boundingBoxCount

accelerationStructureDescriptor.geometryDescriptors = [ geometryDescriptor ]
```

return true false
also return closest intersection distance.  Metal uses this to compute the smallest.

```cpp
struct BoundingBoxResult {
    bool accept [[accept_intersection]];
    float distance [[distance]];
};
```

```cpp
[[intersection(bounding_box)]]
BoundingBoxResult sphereIntersectionFunction(float3 origin            [[origin]],
                                             float3 direction         [[direction]],
                                             float minDistance        [[min_distance]],
                                             float maxDistance        [[max_distance]],
                                             uint primitiveIndex      [[primitive_id]],
                                             device Sphere *spheres   [[buffer(0)]])
{
    float distance;

    if (!intersectRaySphere(origin, direction, spheres[primitiveIndex], &distance))
        return { false, 0.0f };

    if (distance < minDistance || distance > maxDistance)
        return { false, 0.0f };

    return { true, distance };
}
```

ray payload.  Can modify the payload for e.g. outputing a normal.
Keep in mind the changes to the payload will be visible whether you accept the intersection or not.  Usually you only want to modify the payload if you accept the intersection.

```cpp
[[intersection(bounding_box)]]
BoundingBoxResult sphereIntersectionFunction(/* ... */,
                                             ray_data float3 & normal [[payload]])
{
    // ...

    if (distance < minDistance || distance > maxDistance)
        return { false, 0.0f };

    float3 intersectionPoint = origin + direction * distance;
    normal = normalize(intersectionPoint - spheres[primitiveIndex].origin);

    return { true, distance };
}
```

```cpp
[[kernel]]
void rtKernel(/* ... */)
{
    // generate ray, create intersector...

  float3 normal;

    intersection = intersector.intersect(r, accelerationStructure, functionTable, normal);

    // shading...
}
```




## Linking intersection functions

for intersector to work, need to link together with intersection function.

```swift
// Load functions from Metal library
let sphereIntersectionFunction = library.makeFunction(name: “sphereIntersectionFunction”)!
// other functions...

// Attach functions to ray tracing compute pipeline descriptor
let linkedFunctions = MTLLinkedFunctions()

linkedFunctions.functions = [ sphereIntersectionFunction, alphaTestFunction, ... ]

computePipelineDescriptor.linkedFunctions = linkedFunctions

// Compile and link ray tracing compute pipeline
let computePipeline = try device.makeComputePipeline(descriptor: computePipelineDescriptor,
                                                     options: [],
                                                     reflection: nil)
                                             
```


"Intersection function table"

Contains pointers to the executable code in the pipeline state.
Each geometry can have its own intersection function

`intersectionFunctionTableOffset`.  These 2 values are added together to decide which IFT to call.

```swift
class MTLAccelerationStructureGeometryDescriptor : NSObject {

    var intersectionFunctionTableOffset: Int

// ...

}

struct MTLAccelerationStructureInstanceDescriptor {
    var intersectionFunctionTableOffset: UInt32
    // ...
};
```

```swift
// Allocate intersection function table
let descriptor = MTLIntersectionFunctionTableDescriptor()

descriptor.functionCount = intersectionFunctions.count

let functionTable = computePipeline.makeIntersectionFunctionTable(descriptor: descriptor)

for i in 0 ..< intersectionFunctions.count {
    // Get a handle to the linked intersection function in the pipeline state
    let functionHandle = computePipeline.functionHandle(function: intersectionFunctions[i])

    // Insert the function handle into the table
    functionTable.setFunction(functionHandle, index: i)
}

// Bind intersection function resources
functionTable.setBuffer(sphereBuffer, offset: 0, index: 0)
```

Using IFT is very similar to acceleration structure.

```cpp
[[kernel]]
void rtKernel(primitive_acceleration_structure accelerationStructure   [[buffer(0)]],
              intersection_function_table<triangle_data> functionTable [[buffer(1)]],
              /* ... */)
{
    // generate ray, create intersector...

    intersection = intersector.intersect(r, accelerationStructure, functionTable);

    // shading...
}
```
```cpp
encoder.setIntersectionFunctionTable(functionTable, bufferIndex: 1)
```

# Wrap up
* intersection
* acceleration structures
* intersection functions

since a ray could hit any surface at any time, have to be able to hit arbitrary materials on demand.  For more detail on this technique, see

[[Get to know metal function pointers]]


[[Ray tracing with metal - 19]]
[[Metal for ray tracing acceleration - 18]]
