#metal 

# Ray tracing
* Tracing a ray's path as it interacts with a scene
* Applications
	* Rendering
	* Audio and physics simulation
	* collision detection
	* AI and pathfinding

Used in offline rendering to model individual rays of light
* Reflections and refraction
* Shadows
* Ambient occlusion
* Global illumination

# Ray tracing with metal
1.  Compute kernel generates rays
2.  Test for intersections against the geometry in the scene via an intersector and acceleration structure.
3.  Intersection point represents light bouncing on the surface
4.  Compute a color for each intersection and update the image.  Shading.  Can also generate additional rays which are tested for intersection

Repeat until satisfied.

# Ray tracing from render pipelines

Single render pass.  Makes it easy to add raytracing to the renderer.  However, without this support, we need to add a compute pass.  Can add after rendering to update render image.

Adding this extra pass means writing more output to memory.  e.g. render->compute involves memory bandwidth as memory is copied out of the render pass and into the compute pass.

What if I want to use raytracing in the render pass?  Then we have to interleave render and compute passes.  This involves tons of memory bandwidth.

With new support for raytracing from render stages, we don't need to leave our render pass.

## Ray tracing from render pipelines
* build accel structure
* define custom intersection functions

Create stage-specific function tables

## Intersection function tables
We have some solutions (functions) for sphere, cone, torus.

To use the functions, we need to create a function table from the pipeline stage.  With the table, we can create function handles and populate the table.

* Specify intersection functions on render pieline state:

```objc
// Create and attach MTLLinkedFunctions object
NSArray <id <MTLFunction>> *functions = @[ sphere, cone, torus ];

MTLLinkedFunctions *linkedFunctions = [MTLLinkedFunctions linkedFunctions];
linkedFunctions.functions = functions;

pipelineDescriptor.fragmentLinkedFunctions = linkedFunctions;

// Create pipeline
id<MTLRenderPipelineState> rayPipeline;
rayPipeline = [device newRenderPipelineStateWithDescriptor:pipelineDescriptor
                                                     error:&error];
 ```
 
 ```objc
 // Fill out intersection function table descriptor
MTLIntersectionFunctionTableDescriptor *tableDescriptor =
    [MTLIntersectionFunctionTableDescriptor intersectionFunctionTableDescriptor];

tableDescriptor.functionCount = functions.count;

// Create intersection function table
id<MTLIntersectionFunctionTable> table;
table = [rayPipeline newIntersectionFunctionTableWithDescriptor:tableDescriptor
                                                          stage:MTLRenderStageFragment];
```

populate table:

```objc
id<MTLFunctionHandle> handle;

for (NSUInteger i = 0 ; i < functions.count ; i++) {
    // Get a handle to the linked intersection function in the pipeline state
    handle = [rayPipeline functionHandleWithFunction:functions[i]
                                               stage:MTLRenderStageFragment];

    // Insert the function handle into the table
    [table setFunction:handle atIndex:i];
}
```

Now we just have to use this to intersect.

Bind resources.
```objc
[renderEncoder setFragmentAccelerationStructure:accelerationStructure atBufferIndex:0];
[renderEncoder setFragmentIntersectionFunctionTable:table atBufferIndex:1];
```

Intersect
```objc
[[fragment]]
float4 rayFragmentShader(vertex_output vo [[stage_in]],
                         primitive_acceleration_structure accelerationStructure,
                         intersection_function_table<triangle_data> functionTable,
                         /* ... */)
{
    // generate ray, create intersector...

    intersection = intersector.intersect(ray, accelerationStructure, functionTable);

    // shading...
}
```

More details abou thow to prepare your pipline
[[Discover ray tracing with metal]]

* Build acceleration structure
* Create custom intersection function table
* Add intersector to shaders

New opportunities
* Single render pass
* Hybrid rendering
* Tile functions on Apple silicon

[[Modern rendering with Metal]]
[[Explore hybrid rendering with Metal ray tracing]]


# Usability and portability
Portability from other ray tracing APIS.

## Intersection query
* User-driven acceleration structure traversal
* Alternative to intersector for simple scenarios
* In-line custom intersection testing

## Alpha testing
Can customize an intersector.  Triangle intersector is responsible for accepting/rejecting intersections.

1.  call `intersect()`
2.  Acceleration structure is traversed to find an intersection
3.  Within the intersector, intersection function is called.
4.  Accept/reject based on intersection function logic.

This is a great programming model because it's performant and convenient.  But it does require creating a new intersection function and linking it.

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

With intersection query, we can place this inline, without the need of the intersection function.

1.  Start intersection process
2.  Query object contains state of the traversal and result.
3.  Each time it intersects a custom primitive or "non-opaque triangle", control is returned to the shader for your to evaluate the intersection candidates.
4.  If it passes your logic, you commit it to update the current intersection
5.  Then continue the intersection process.
6.  If fails intersection logic, you can ignore and continue.

Alpha testing example

```cpp
intersection_query<instancing, triangle_data> iq(ray, as, params);

// Step 1: start traversing acceleration structure
while (iq.next())
{
    // Step 2: candidate was found. Check type and run custom intersection. 
    switch (iq.get_candidate_intersection_type())
    {
	    case intersection_type::triangle:
	    { 
			//note that this code is now inline,
			//rather than in a separate function
	       bool alphaTestResult = alphaTest(iq.get_candidate_geometry_id(),
	                                iq.get_candidate_primitive_id(),
	                                iq.get_candidate_triangle_barycentric_coord());
    	   // Step 3: commit candidate or ignore
           if (alphaTestResult) 
               iq.commit_triangle_intersection()
  	  }
    }
}
```

Now to do shading, you need to query this information.

```cpp
switch (iq.get_committed_intersection_type())
{
  // Miss case  
  case intersection_type::none:
  {
      missShading();
      break;
  } 
  
  // Triangle intersection was committed. Query some info and do shading.
  case intersection_type::triangle:
  {
      shadeHitTriangle(iq.get_committed_instance_id(),
                       iq.get_committed_distance(),
                       iq.get_committed_triangle_barycentric_coord());
      break;
  }
}
```

Some things to consider. 

|                                   | Intersector                             | Intersection query                              |
|-----------------------------------|-----------------------------------------|-------------------------------------------------|
| Existing code                     | Intersector in compute on apple silicon | Existing quer code                              |
| Complexity of custom intersection | Intersection functions and tables       | In line custom intersection                     |
| Performance                       | Encapsulates all intersection work      | Custom intersection control switching to shader |

## Acceleration structure data
* User instance IDs
* Instance transforms

We have multiple instances.  Underneath this, we have an acceleration structure with a set of nodes that branch until you reach the instances.  Instances are at lowest level.  Currently, when you intersect an instance, you only get the instance ID from the intersection results.

With this, you could maintain your own table of data, but there's data we can expose to help you.

## User-defined instance ID
32-bit per-instance value
Get this from intersection results
Use for indices or encode custom data

Reuse user IDs to encode a custom color for each instance.

```cpp
// New instance descriptor type

typedef struct {
    uint32_t userID;
    // Members from MTLAccelerationStructureInstanceDescriptor...
} MTLAccelerationStructureUserIDInstanceDescriptor;

// Specify instance descriptor type through acceleration structure descriptor

accelDesc.instanceDescriptorType = MTLAccelerationStructureInstanceDescriptorTypeUserID;
```

Retrieving
```cpp
// Available in intersection functions

[[intersection(bounding_box, instancing)]]
IntersectionResult sphereInstanceIntersectionFunction(unsigned int userID[[user_instance_id]],
                                                      /** other args **/)
{
    // ...
}
```

Retrieving from intersection results

```cpp
// Available from intersection result

intersection_result<instancing> intersection = instanceIntersector.intersect(/* args */);

if (intersection.type != intersection_type::none)
    instanceIndex = intersection.user_instance_id;

// Available from intersection query

intersection_query<instancing> iq(/* args */);

iq.next()

if (iq.get_committed_intersection_type() != intersection_type::none)
    instanceIndex = iq.get_committed_user_instance_id();
```

## Instance transforms
Just like user instance ID, we added support for instance transforms.

* Intersection functions
* Intersection results

```cpp
// Available in intersection functions

[[intersection(bounding_box, instancing, world_space_data)]]
IntersectionResult intersectionFunction(float4x3 objToWorld [[object_to_world_transform]],
                                        float4x3 worldToObj [[world_to_object_transform]],
                                        /** other args **/)
{
    // ...
}
```

```cpp
// Available from intersection result

intersection_result<instancing, world_space_data> result = 
    intersector.intersect(/* args */);

if (result.type != intersection_type::none) {
    output.myObjectToWorldTransform = result.object_to_world_transform;
    output.myWorldToObjectTransform = result.world_to_object_transform;
}
```

```cpp
// Available from intersection query

intersection_query<instancing> iq(/* args */);

iq.next()

if(iq.get_committed_intersection_type() != intersection_type::none){
    output.myObjectToWorldTransform = iq.get_committed_object_to_world_transform();
    output.myWorldToObjectTransform = iq.get_committed_world_to_object_transform();
}
```


## Usability and portability
* Intersection query
* user instance IDs
* instance transforms


# Production rendering

## Extended limits
Some user started hitting internal limits of acceleration structures, particularly at production-scale
* Support for data at production scale
* Balance
	* Acceleration structure size
	* Limits of internal identifiers
	* Performance

This is an opt-in feature.  

| Limit                                          | current value | extended value |
|------------------------------------------------|---------------|----------------|
| Primitives in primitive acceleration structure | 2^28          | 2^30           |
| Geometries in primitive acceleration structure | 2^24          | 2^30           |
| Instances in instance acceleration structure   | 2^24          | 2^30           |
| Visibility mask bits                           | 8             | 32             |

Specify through descriptor

```cpp
// Specify through acceleration structure descriptor

accelDesc.usage = MTLAccelerationStructureUsageExtendedLimits;

// Specify intersector tag

intersector<extended_limits> extendedIntersector;
```

## Motion blur
We often assume camera exposure is instantaneous.  In real life, camera exposure lasts for a non-zero period of time.

This effect can go a long way toward making computer-generated images more realistic.

Games often approximate this in screenspace.  But raytracing allows us to simulate physically-accurate blur.  Even indirect effects, like shadows and reflections.

Most raytracing applications randomly sample physical dimensions such as incoming light.  To add motion blur, we can just choose a random time for each ray as well.

Metal will intersect the scene.  

As we accumulate more and more samples we start to converge on a motion blur image.  You can implement this today using custom intersection function.  You can compute the bounding boxes of primitive throughout the exposure.  However, this would be inefficient, the bounding boxes would be so large that some rays would need to check intersection with primitives that they won't intersect.

Instead, metal has builtin support.  Efficiently handle these case.

```cpp
// Randomly sample time

float time = random(exposure_start, exposure_end);

result = intersector.intersect(ray, acceleration_structure, time);
```

Animation created with keyframes.  Uniformly distributed between the start/end of the animation.

As rays traverse the accel structure, they can fetch data from any keyframe based on their time value.  

both instances and primitive motion.  Primitive is needed to do deformation.

Note that both instance and primitive animation are based on keyframe animation.

### instance motion
Each instance is associated with a transformation matrix.  Where to place geometry in the scene.

To animate the sphere, we provide two transformation matrices representing start and endpoint of animation.

Metal interpolates based on time parameter.  Specific example using two keyframes.  Metal supports an arbitrary number of keyframes.

Standard metal instance descriptor only has room for a single transformation matrix.

### motion instance descriptor
Start index and count into transform buffer.  I think the latter has keyframes?

```objc
descriptor = [MTLInstanceAccelerationStructureDescriptor new];

descriptor.instanceDescriptorType = MTLAccelerationStructureInstanceDescriptorTypeMotion;

// Buffer containing motion instance descriptors
descriptor.instanceDescriptorBuffer = instanceBuffer;
descriptor.instanceCount = instanceCount;

// Buffer containing MTLPackedFloat4x3 transformation matrices
descriptor.motionTransformBuffer = transformsBuffer;
descriptor.motionTransformCount = transformCount;

descriptor.instancedAccelerationStructures = primitiveAccelerationStructures;
```

Specify intersector tag
```cpp
// Specify intersector tag

kernel void raytracingKernel(acceleration_structure<instancing, instance_motion> as,
                             /* other args */)
{
    intersector<instancing, instance_motion> intersector;

    // ...
}
```

Tells intersector to expect an accel structure that has instance motion.  That's all we need for instance motion.

### primitive motion
Each primitive can move separately.  e.g. skin.

Need to provide separate 3d model for each keyframe and metal can interpolate between them.

vertex data for each keyframe.

Collect keyframe vertex buffers
```objc
// Collect keyframe vertex buffers

NSMutableArray<MTLMotionKeyframeData*> *vertexBuffers = [NSMutableArray new];

for (NSUInteger i = 0 ; i < keyframeBuffers.count ; i++) {
    MTLMotionKeyframeData *keyframeData = [MTLMotionKeyframeData data];

    keyframeData.buffer = keyframeBuffers[i];

    [vertexBuffers addObject:keyframeData];
}
```

Create motion geometry descriptor
```objc
// Create motion geometry descriptor

MTLAccelerationStructureMotionTriangleGeometryDescriptor *geometryDescriptor =
    [MTLAccelerationStructureMotionTriangleGeometryDescriptor descriptor];

geometryDescriptor.vertexBuffers = vertexBuffers;
geometryDescriptor.triangleCount = triangleCount;
```

Instead of providing a single vtx buffer, provide an array

Create acceleration structure descriptor

```objc
// Create acceleration structure descriptor

MTLPrimitiveAccelerationStructureDescriptor *primitiveDescriptor =
    [MTLPrimitiveAccelerationStructureDescriptor descriptor];

primitiveDescriptor.geometryDescriptors = @[ geometryDescriptor ];

primitiveDescriptor.motionKeyframeCount = keyframeCount;
```

Specify intersector tag:
```cpp
// Specify intersector tag

kernel void raytracingKernel(acceleration_structure<primitive_motion> as,
                             /* other args */)
{
    intersector<primitive_motion> intersector;

    // ...
}
```

You can actually use both types of animation at the same time.

We've been putting in a lot of work.  We can't wait to see the amazing content you create.