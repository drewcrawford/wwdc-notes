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

`primitive_acceleration_structure`

# Acceleration structure

Recursively partition space so we could eliminate triangles that cannot possibly intersect a given ray
Metal builds this for us

1.  Create descriptor
2.  Allocate storage
3.  Build

Primitive structures, instance structures

## Allocation stuff

`device.accelerationStructureSizes`
`device.makeAccelerationStructure`
allocate a scratch buffer as well.  This is used during the accel init.

## Building
`comamndEncoder.build(accleerationSTructure:)`

Builds now run entirely on the GPU timeline with no CPU synchronization.

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

Also can write bounding-box intersection functions
Invoked when ray intersects a user-provided bounding box
models custom primitives.

evidently they have a bezier curve bounding box intersection function

## Sphere example
1.  Axis-aligned bounding boxes
2.  Acceleration structure over the elements
3.  Invokes filter function each time it finds an intersection

The functions
1.  Accept or reject intersection
2.  Return closest intersection distance

`[[intersection(bounding_box)]]`

```c
struct boudningBoxResult {
bool accept [[accept_intersection]]
float distance ...
}
```

```ray_data float3 & normal [[payload]]```

Keep in mind the changes to the payload will be visible whether you accept the intersection or not.

## Linking intersection functions

```swift
library.makeFnction()
let linkedfunction = MTLLinkedfunctions()
linkedFunctions.functions = [f1,f2]
computePipelineDescriptor.linkedFunctions = linkedFunctions
```

"Intersection function table"

Contains pointsers to the executable code in the pipeline state.
Each geometry can have its own intersection function

`intersectionFunctionTableOffset`.  These 2 values are added together to decide which IFT to call.

`MTLIntersectionFunctionTableDescriptor()` 
created from `computePipeline.makeIntersectionFunctionTable()`.  Note this is specific to the PSO.
`computePipeline.functionHandle(function: ...)`
`functionTable.setFunction(functionHandle, index)`

[[Get to know metal function pointers]]


[[Ray tracing with metal - 19]]
[[Metal for ray tracing acceleration - 18]]
