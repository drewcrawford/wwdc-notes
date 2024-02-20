Discover how you can enhance the visual quality of your games and apps with Metal ray tracing. We'll take you through the fundamentals of the Metal ray tracing API. Explore the latest enhancements and techniques that will enable you to create larger and more complex scenes, reduce memory usage and build times, and efficiently render visual content like hair and fur.

1.  Define geometry
2. acceleration structure

# Build your scene

acceleration structure
1.  Descriptor that provides geometry
2. allocate structure
3. build structure

descriptor contains one or more geometry descriptor.  3 types.
* triangles
* bounding boxes (custom intersection function)
* curves

### 3:06 - Create triangle geometry descriptor
```swift
// Create geometry descriptor:
let geometryDescriptor = MTLAccelerationStructureTriangleGeometryDescriptor()

geometryDescriptor.vertexBuffer = vertexBuffer
geometryDescriptor.indexBuffer = indexBuffer
geometryDescriptor.triangleCount = triangleCount
```

 ### 3:20 - Create bounding box geometry descriptor
```swift
// Create geometry descriptor:
let geometryDescriptor = MTLAccelerationStructureBoundingBoxGeometryDescriptor()

geometryDescriptor.boundingBoxBuffer = boundingBoxBuffer
geometryDescriptor.boundingBoxCount = boundingBoxCount
```

For more details on intersection function, see [[Discover ray tracing with metal]]

Hair, fur, vegetation can have many primitives.  New curve primtive.  Perfectly smooth.  Compact memory footprint.  Faster acceleration structure builds.

Curve is made of connected curve segments.  Each segment is a unique primitive

Segments defined by control points.  Interpolated by basis functions.  2,3,4 control points per segment.
4 different curve basis
* bezier
* catmull-rom
* b-spline
* linear

Control point index buffer.  

To render curves, they need some kind of 3d shape.  Each control poitn has a radius which is interpolated along the length of the curve.  By default, they are rendered by 3d cylindrical cross-section.  Great for closeup.

For far away, we support flat curves.  When you don't need full 3d geometry.

Similar to triangles, curve geometry is represented with curve geometry descriptor.


### 6:42 - Create curve geometry descriptor
```swift
let geometryDescriptor = MTLAccelerationStructureCurveGeometryDescriptor()
  
geometryDescriptor.controlPointBuffer = controlPointBuffer
geometryDescriptor.radiusBuffer = radiusBuffer
geometryDescriptor.indexBuffer = indexBuffer

geometryDescriptor.controlPointCount = controlPointCount
geometryDescriptor.segmentCount = segmentCount
geometryDescriptor.curveType = .round
geometryDescriptor.curveBasis = .bezier
geometryDescriptor.segmentControlPointCount = 4
```

Acceleration structure descriptor.


### 7:29 - Create primitive acceleration structure descriptor
```swift
// Create acceleration structure descriptor
let accelerationStructureDescriptor = MTLPrimitiveAccelerationStructureDescriptor()

// Add geometry descriptor to acceleration structure descriptor
accelerationStructureDescriptor.geometryDescriptors = [ geometryDescriptor ]
```

Now that we've described, we can allocate.  Metal gives you full control over when/where this memory is allocated.  This is a 2-part operation.  First, calculate the size of the objects needed for the build.

Allocating from heap will let you reduce overhead later.  Heap may have additional size/alignment requirements.


### 8:08 - Query for acceleration size and alignment requirements
```swift
// Query for acceleration structure sizes
let sizes: MTLAccelerationStructureSizes
sizes = device.accelerationStructureSizes(descriptor: accelerationStructureDescriptor)

// Query for size and alignment requirement in a heap
let heapSize: MTLSizeAndAlign
heapSize = device.heapAccelerationStructureSizeAndAlign(size: sizes.accelerationStructureSize)
```

scratch memory - only accessed by GPU.  Can use private storage mode buffer.
### 8:39 - Allocate acceleration structure and scratch buffer
```swift
// Allocate acceleration structure from heap
var accelerationStructure: MTLAccelerationStructure!
accelerationStructure = heap.makeAccelerationStructure(size: heapSize.size)

// Allocate scratch buffer
let scratchBuffer = device.makeBuffer(length: sizes.buildScratchBufferSize,
                                      options: .storageModePrivate)!
```

Now we build.  Schedule build operation and metal will build acceleration for yu on-gpu.

Acceleration structure command encoder.  This encoder has several methods.

In this case we call `build`.  

### 8:40 - Encode the acceleration structure build
```swift
let commandEncoder = commandBuffer.makeAccelerationStructureCommandEncoder()!

commandEncoder.build(accelerationStructure: accelerationStructure,
                     descriptor: accelerationStructureDescriptor,
                     scratchBuffer: scratchBuffer,
                     scratchBufferOffset: 0)

commandEncoder.endEncoding()
```
# Scale with instancing

metal supports instance acceleration structures.  It would take an enormous amount of memory to store a complex detailed environment in a single primitive acceleration structure.  But this scene has a repetitive structure.

All unique objects can be represented as primitive acceleration structures.  These can be combined into an instance structure representing the whole scene.  So, while a primitive structure contains geometry, instance contains references to other accel structures transformed to various positions, sizes, etc.

1.  Create descriptor
2. configure instances
3. build


### 11:30 - Create instance acceleration structure descriptor
```swift
var instanceASDesc = MTLInstanceAccelerationStructureDescriptor()

instanceASDesc.instanceCount = ...
instanceASDesc.instancedAccelerationStructures = [ mountainAS, treeAS, ... ]
instanceASDesc.instanceDescriptorType = .userID
```

First we allocate a buffer per-intance data.  Size depends on number of isntances and size of each instance descriptor.  Allocate like any other buffer.



### 12:07 - Allocate the instance descriptor buffer
```swift
let size = MemoryLayout<MTLAccelerationStructureUserIDInstanceDescriptor>.stride
let instanceDescriptorBufferSize = size * instanceASDesc.instanceCount

let instanceDescriptorBuffer = device.makeBuffer(length: instanceDescriptorBufferSize,
                                                 options: .storageModeShared)!
    
instanceASDesc.instanceDescriptorBuffer = instanceDescriptorBuffer
```

Now we fill instance buffer.   Create a descriptor each instance.  Acceleration structure index.  Also have a transformation matrix, mask, etc.

### 12:33 - Populate instance descriptors
```swift
var instanceDesc = MTLAccelerationStructureUserIDInstanceDescriptor()

instanceDesc.accelerationStructureIndex = 0    // index into instancedAccelerationStructures
instanceDesc.transformationMatrix = ...
instanceDesc.mask = 0xFFFFFFFF
```

Now we build the structure.  Same as for primitives.

All steps before build can run on CPU, but if too many instances, this may be comptue intensive.  Since instance descriptors are stored in a metal buffer we can do this on-gpu instead.  But if you want to do something like instance culling, you have to cull on CPU to find instance count.

new this year, can drive this on GPU, with new indirect structure descriptor.  Now cull instances on GPU.

To configure indirect instance acceleration structure descriptor:

### 14:06 - Configure indirect instance acceleration structure descriptor
```swift
var instanceASDesc = MTLIndirectInstanceAccelerationStructureDescriptor()

instanceASDesc.instanceDescriptorType = .indirect
instanceASDesc.maxInstanceCount = ...
instanceASDesc.instanceCountBuffer = ...
instanceASDesc.instanceDescriptorBuffer = ...
```

you can identify structure being instanced from MSL.
### 14:29 - Populate indirect instance descriptor
```swift
device MTLIndirectAccelerationStructureInstanceDescriptor *instance_buffer = ...;
// ...
acceleration_structure<> as = ...;
instance_buffer[i].accelerationStructureID = as;
instance_buffer[i].transformationMatrix[0] = ...;
instance_buffer[i].transformationMatrix[1] = ...;
instance_buffer[i].transformationMatrix[2] = ...;
instance_buffer[i].transformationMatrix[3] = ...;
instance_buffer[i].mask = 0xFFFFFFFF;
```

So far we've covered 2-level instancing.  Now you can take advantage of multilevel instancing.  An instance acceleration structure can contain not just primitives but also other instance accel structures.

ex scene contains tree contains leaves.

Multilevel instancing isn't just for production renderers.  Valuable for realtime apps like games.  Building worlds from instances of game objects.  However, games are different from production renderers.

Games use long lists of instances for game objects.  Games also rebuild their instance acceleration structure each frame and high instance counts means a lot of gpu time for rebuild.

However in a game, much of the content is static and isn't updated per-frame.  Can split world into static and dynamic structures to limit updates.

Balance hierarchy depth and traversal time.  Using 3 levels of instancing lets you reduce build tiem with only minor impact on trace time, overall reducing frame time.

Optimize acceleration structures:
* build parallelization
	* Typical app will build/update many accel structures.  Can greatly reduce startup time by running these builds in parallel.  Encode multiple builds with the same command encoder so they can run in parallel.  Want to parallelize as many builds as you can while ensuring that the working set fits in memory.  
	* Re-use scratch buffers from one batch of builds to the next
* refitting
	* Avoid rebuilding altogether.  When metal builds a structure, it groups nearby primitives into a hierarchy.  If primitives move, those boxes no longer represent the scene.  But if the geometry only changes slightly, hierarchy may still be reasonable.  Instead of building from scratch, metal can refit accel structure quickly.  Cheaper than rebuilding from scratch.
	* refit in-place or onto new structure
* compaction



 
### 19:22 - Update geometry using refitting
```swift
// Allocate scratch buffer
let scratchBuffer = device.makeBuffer(length: sizes.refitScratchBufferSize,
                                      options: .storageModePrivate)!

// Create command buffer/encoder ...

// Refit acceleration structure
commandEncoder.refit(sourceAccelerationStructure: accelerationStructure,
                     descriptor: asDescriptor,
                     destinationAccelerationStructure: accelerationStructure,
                     scratchBuffer: scratchBuffer,
                     scratchBufferOffset: 0)
```

### compaction.

When you first build an accel structure, metal can't know exactly how much memory it needs, uses conservative estimate.  After build, metal can calculate minimum size needed to represent it.

With compaction we can allocate a new structure with minimum size, use the GPU to copy from current structure to new one.  Especially valuable for primitive accel structure.

First find the size, and then copyAndCompact.

[[Maximize your Metal ray tracing performance]]

### 20:24 - Use compaction to reclaim memory
```swift
// Use compaction to reclaim memory

// Create command buffer/encoder ...

sizeCommandEncoder.writeCompactedSize(accelerationStructure: accelerationStructure,
                                      buffer: sizeBuffer,
                                      offset: 0,
                                      sizeDataType: .ulong)

// endEncoding(), commit command buffer and wait until completed ...

// Allocate new acceleration structure using UInt64 from sizeBuffer ...

compactCommandEncoder.copyAndCompact(sourceAccelerationStructure: accelerationStructure,
                             destinationAccelerationStructure: compactedAccelerationStructure)
```

# Intersect rays

Bind your acceleration structure on command encoder.  Now you can intersect rays with this structure in your GPU function.  Declare the function with an acceleration structure parameter.

### 21:36 - Set acceleration structure on the command encoder
```swift
encoder.setAccelerationStructure(primitiveAccelerationStructure, bufferIndex:0)
```

### 21:48 - Intersect rays with primitive acceleration structure
```swift
// Intersect rays with a primitive acceleration structure

[[kernel]]
void trace_rays(acceleration_structure<> as, /* ... */) {
  intersector<> i;

  ray r(origin, direction);

  intersection_result<> result = i.intersect(r, as);

  if (result.type == intersection_type::triangle) {
    float distance = result.distance;


    // shade triangle...
  }
}
```

### 22:24 - Use triangle_data tag to get triangle barycentric coordinates
```cpp
// Intersect rays with a primitive acceleration structure

[[kernel]]
void trace_rays(acceleration_structure<> as, /* ... */) {
  intersector<triangle_data> i;

  ray r(origin, direction);

  intersection_result<triangle_data> result = i.intersect(r, as);

  if (result.type == intersection_type::triangle) {
    float distance = result.distance;
    float2 coords = result.triangle_barycentric_coord;

    // shade triangle...
  }
}
```

What about with instancing though?  

### 22:51 - Set instance acceleration structure on the command encoder
```swift
encoder.setAccelerationStructure(instanceAccelerationStructure, bufferIndex:0)
encoder.useHeap(accelerationStructureHeap);
```

need `instancing` and `max_levels` tag.


### 23:07 - Intersect rays with instance acceleration structure
```cpp
// Intersect rays with an instance acceleration structure

[[kernel]]
void trace_rays(acceleration_structure<instancing> as, /* ... */) {
  intersector<instancing, max_levels<3>> i;

  ray r(origin, direction);

  intersection_result<instancing, max_levels<3>> result = i.intersect(r, as);

  if (result.type == intersection_type::triangle) {
    float distance = result.distance;

    // shade triangle...
  }
}
```

1 - whole scene
2 -> coral, trees, terrain
3 -> parts of trees, like leaves etc.

### 24:43 - Find intersected instance information in the intersection result
```cpp
// Intersect rays with an instance acceleration structure

[[kernel]]
void trace_rays(acceleration_structure<instancing> as, /* ... */) {
  intersector<instancing, max_levels<3>> i;

  ray r(origin, direction);

  intersection_result<instancing, max_levels<3>> result = i.intersect(r, as);

  if (result.type == intersection_type::triangle) {
    float distance = result.distance;
    for (uint i = 0; i < result.instance_count; ++i) {
      uint id = result.instance_id[i];
      // ...
    }
    // shade triangle...
  }
}
```

Few things to keep in mind with curve primitives.

By default, metal assumes you are not using curve primitives.  You can tell metal that you are, by setting geometry type on intersector object.

If you use curve data tag, than intersection result also contains the curve parameter.  Can compute points by plugging into curve basis function.  Learn more in MSL.

### 25:02 - Intersect rays with curve primitives
```cpp
// Intersect rays with curve primitives

[[kernel]]
void trace_rays(acceleration_structure<> as, /* ... */) {
  intersector<> i;

  i.assume_geometry_type(geometry_type::curve | geometry_type::triangle);

  ray r(origin, direction);

  intersection_result<> result = i.intersect(r, as);

  if (result.type == intersection_type::curve) {
    float distance = result.distance;
    // shade curve...
  }
}
```
### 25:26 - Find curve parameter in the intersection result
```cpp
// Intersect rays with curve primitives

[[kernel]]
void trace_rays(acceleration_structure<> as, /* ... */) {
  intersector<curve_data> i;

  i.assume_geometry_type(geometry_type::curve | geometry_type::triangle);

  ray r(origin, direction);

  intersection_result<curve_data> result = i.intersect(r, as);

  if (result.type == intersection_type::curve) {
    float distance = result.distance;
    float param = result.curve_parameter;
    // shade curve...
  }
}
```


You can also assume certain curve types to improve performance.

### 26:04 - Set geometry type on the intersector for better performance
```swift
// Intersect rays with curve primitives

[[kernel]]
void trace_rays(acceleration_structure<> as, /* ... */) {
  intersector<curve_data> i;

  i.assume_geometry_type(geometry_type::curve | geometry_type::triangle);
  i.assume_curve_type(curve_type::round);
  i.assume_curve_basis(curve_basis::bezier);
  i.assume_curve_control_point_count(3);

  ray r(origin, direction);

  intersection_result<curve_data> result = i.intersect(r, as);

  if (result.type == intersection_type::curve) {
    float distance = result.distance;
    float param = result.curve_parameter;
    // shade curve...
  }
}```


# Debug and profile


## Shader validation
catches issues which may lead to crashes/corruption.

* entire Metal API coverage
## Accelerated structure viewer

inspect the scene which you use for intersection testing.  When I open structure viewer, I get an outline on the left for navigating individual building blocks, down to geometry primitives.

highlighting mode.  ex AABB have deep traversals, corresponding to more expensive intersection testing.

acceleration structure highlight mode.  visualizes each structure in different colors.


## shader debugger

Helps with troubleshooting issues.  Here I'm looking at a compute dispatch.  To begin debugging the shader, I can choose the debug button, etc.  After it finishes gathering data, I can examine the value for each variable at any point during shader execution.

Take a closer look at value for primitive ID, debugger also gives me data from neighboring threads.

## profiling timeline

In addition, change debug navigator to view all pipeline states in the workload.  Most expensive states at the top.  Further expanding a pipeline state reveals the shader code.  Get per-line shading profiling insights.  

Support all of metal's new raytracing features.

Raytracing also supports many more features such as primitiveinstance motion, custom intersection functions, intersection_query, etc.

# Wrap up

* build your scene
* Scale with instancing
* Intersect rays

[[Maximize your Metal ray tracing performance]]
[[Enhance your app with Metal ray tracing]]
[[Discover ray tracing with metal]]







* https://developer.apple.com/documentation/metal/metal_sample_code_library/rendering_reflections_in_real_time_using_ray_tracing
* https://developer.apple.com/documentation/metal/metal_sample_code_library/accelerating_ray_tracing_using_metal
* https://developer.apple.com/documentation/metal/metal_sample_code_library/accelerating_ray_tracing_using_metal
