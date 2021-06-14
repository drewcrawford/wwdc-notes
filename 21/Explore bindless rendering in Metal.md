#metal 

Bindless is a modern resource binding model that allows making groups of resources available on the GPU

# Motivation
Argument buffer encoding

Suppose we have a ray tracing kernel targeting (finding intersections) some acceleration structure.

Find objects between intersection point and the light.  No object attributes are needed, beyond workspace position.

For other FX, this gets more complicated.  

```cpp
if (intersection.type == intersection_type::triangle) 
{
  // solid blue triangle
  color = float4(0.0f, 0.0f, 1.0f, 1.0f);
}   

outImage.write(color, tid);
```

If we paint a solid color, the reflection will not look accurate.  Need to determine the attributes of each reflected point found.

From our shader, we potentially need the entire scene data.
* Buffers and materials

Unable to bind all possible resources directly.

## Bindless binding model


* Aggregate resources
* Resources indirectly available to shaders
* Argument buffers tier 2
	* Apple6 (A13), Mac2 families
	* Compute, vertex, fragment, tile
	* Raytracing and rasterization

Advantages
* Removes slot limits on bound resources
* Nice optimization opportunities we will explore later.

In metal 2, we introduced argument buffers.  Idea behind bindless model is to diverge this capability.  Making them available to GPU at the same time.

## ex
* textures
* vertex data
* index data

In our case, because we want to make all textures available at once, we need to aggregate.  First create a meshes buffer.  This buffer allows referencing vertex and index into the scene.

Material owns textures.

Instance owns mesh and material.  

Encode all instances as array.

with this, we have the full scene and resources encoded/linked with argument buffers.  When we want to reference any resources, we just need a pointer to the instances buffer.

## Residency of indirect resources

Because we pass a pointer, metal knows the reference, but not what we access indirectly.

* App responsible for declaring residency
* Signaling to the driver to make its memory available to GPU.
* `useResource:usage:`
* `useResource:usage:stages:`

Failing to do this is a common cause of GPU restarts.  Otherwise memory pages may not be present.

 Heaps.  We recommend you use them for best performance.  There are a few considerations.
 
 * Read-only or read-write
	 * If GPU needs to write, need to declare with right usage flag
	 * Any resources that may have been modified that we intend to read from, need their own `useResource` so we can flush caches etc.
 * Resource tracking.  Since metal 2.3, metal can be configured to track (heaps)?
	 * However, since heaps are a single resource, handled at the heap level.  

### tracked heap

render pass A writes => heap => texture
render pass B reads => heap => distinct texture

A/B may be executed in parallel.  However, due to the potential hazard at the heap level, metal has to serialize to ensure there are no race conditions.  This can potentially increase execution time.

If we know individual resources are independent, this fence can be avoided.

* Separate heaps
* Untracked heap.  You synchronize yourself.

Metal allows us to specify fences at stage granularity.  So we can run vtx and frag concurrently and only block later if it happens to depend on a previoius pass.  Do this if possible.

### Read-only data is easy

Place this in a heap when the app starts etc.  This way you can later make it resident in a single call with minimal overhead.

# Argument buffer encoding
## Encoder creation
```cpp
struct Instance
{
    constant Mesh*     pMesh            [[id(0)]];
    constant Material* pMaterial        [[id(1)]];
    constant float4x4  modelTransform   [[id(2)]];
};
```

Local to world-space transformation

Encoding performed via arg buffer.  

 ```cpp
 // Shader code references scene
kernel void RTReflections( constant Scene* pScene [[buffer(0)]] );
```

`newArgumentEncoder(index)`.    Can ask metal to create encoder for us.  But when we're encoding an entire scene, not all enocders can be reflected.  e.g., the function signature does not know about indirectly referenced buffers (I think this would require control-flow analysis?)

It might not be convenient.

 Evidently arrays are not supported?
 
 For these cases, there's a second mechanism to use a metal argument descriptor `MTLArgumentDescriptor` to create encoders without a `MTLFunction`.
 
 Specify data type and binding index.  Take our descriptors and pass them to `MTLDevice`. 
 
 ```objc
 MTLArgumentDescriptor* meshArg 
= [MTLArgumentDescriptor argumentDescriptor];

meshArg.index    = 0;
meshArg.dataType = MTLDataTypePointer;
meshArg.access   = MTLArgumentAccessReadOnly;

// Declare all other arguments (material and transform)
   
id<MTLArgumentEncoder> instanceEncoder 
= [device newArgumentEncoderWithArguments:@[meshArg, 
                                            materialArg, 
                                            transformArg]];
```


Now we call
```
setArgumentBuffer: scene offset:0
setBuffer: mesh atIndex:0
```

Array encoding
```
setArgumentBuffer:scene offset:(i * encodedLength)
setBuffer:mesh atIndex:0
```

Then just increment `i`

No special treatment is needed on shader side to index into arrays.  Shader does not need to know length of the buffer and can freely index into the array.

# Buffer navigation

* Bindless natural fit for ray tracing
* Bind root of the bindless scene
* In kernel, we trace ray
* Intersection result describes the navigation
	* `instance_id`
	* `geometry_id`
	* `primitive_id`

Your mileage may vary

1.  Find intersection `auto inter = intersector.intersect)`
2.  `inter.instance_id` to find instance in scene.
3.  `inter.geometry_id` to find mesh in instance.
4.  `inter.primitive_id` to find primitive in mesh

```cpp
// Instance and Mesh

constant Instance& instance = pScene->instances[intersection.instance_id];
constant Mesh&     mesh     = instance.mesh[intersection.geometry_id];

// Primitive indices

ushort3 indices; // assuming 16-bit indices, use uint3 for 32-bit

indices.x = mesh.indices[ intersection.primitive_id * 3 + 0 ];
indices.y = mesh.indices[ intersection.primitive_id * 3 + 1 ];
indices.z = mesh.indices[ intersection.primitive_id * 3 + 2 ];
```

```cpp
// Vertex data

packed_float3 n0 = mesh.normals[ indices.x ];
packed_float3 n1 = mesh.normals[ indices.y ];
packed_float3 n2 = mesh.normals[ indices.z ];

// Interpolate attributes

float3 barycentrics = calculateBarycentrics(intersection);
float3 normal       = weightedSum(n0, n1, n2, barycentrics);
```

Now we can shade our reflection

```cpp
if(i.type == intersection_type::triangle)
{
  constant Instance& inst     = get_instance(i);
  constant Mesh&     mesh     = get_mesh(inst, i);
  constant Material& material = get_material(inst, i);
  
  color = shade_pixel(mesh, material, i);
}   

outImage.write(color, tid);
```

Releasing a companion code sample which shows an implementation of all this.  Loaded with model I/O.

Shows how to find intersections and correctly shade them directly from vertexing shaders.

Sample allows direct visualize of reflection.  Great for experimenting with reflection algorithm.

We can apply the same principles to rasterization

# PBR
* Multiple bindings
* Albedo
* Roughness
* Metallic
* Ambient occlusion

* Single binding
* Argument buffer navigation

This arch provides an excellent opportunity to use instance rendering.  Remember to register all textures you access.

In traditional model, you need to reference each texture explicitly.

```cpp
fragment half4 pbrFragment(ColorInOut in [[stage_in]],
                           texture2d< float > albedo    [[texture(0)]],
                           texture2d< float > roughness [[texture(1)]],
                           texture2d< float > metallic  [[texture(2)]],
                           texture2d< float > occlusion [[texture(3)]])
{	
	half4 color = calculateShading(in, albedo, roughness, metallic, occlusion);
	
  return color;
}
```

when using a bindless model, can just pass our root buffer.

```cpp
fragment half4 pbrFragmentBindless(ColorInOut in [[stage_in]], 
                                   device const Scene* pScene [[buffer(0)]])
{
	device const Instance& instance = pScene->instances[in.instance_id];
	device const Material& material = pScene->materials[instance.material_id];
	
	half4 color = calculateShading(in, material);
	
	return color;
}
```

To recap

# Wrap up
* Bindless
* Extremely flexible
* Ray tracing and rasterization
* Indirect pipelines


