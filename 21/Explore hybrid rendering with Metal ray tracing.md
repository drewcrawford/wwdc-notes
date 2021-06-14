#metal 

# Why ray trace
Limitations to what we can achieve with rasterization.

Reflections.  When we're shading a rasterized pixel, we have no context of the rest of the scene for accurate reflections.  Raytracing allows us to trace a ray from the pixel being shaded to see what's out there in the world.

Shadows.  With rasterization, shadow is blurry.  Raytraced shadows are sharper and address aliasing issues without artificial parameters.  Soft shadows can be approximated more accurately.  

Transparency.  Very hard to handle accurately with rasterization.  With raytracing we can create a custom intersection function for transparent materials to define projected shadows.  All shadows look sharper overall.

Why is it that raytracing works?

## Traditional rasterization
1.  Vertex shader
2.  rasterizer
3.  fragment shader
4.  blended into output.

Each pixel independently.  However, when we do shading, we've lost the context of our scene.

Advanced game engines make up with extra passes e.g. SSAO, SSR, shadow maps, etc.

Fragment shader leverages that data.

1.  Rasterize to intermediate textures, e.g. g-buffer.  albedo, depth, normal.
2.  Light approximation.  Smart tricks to approximate light in the scene.  SSAO, reflection.
3.  Intermediate passes are denoised or blurred for a smoother image.

While these techniques can improve the image, they're just an approximation

## Raytracing
Tracing a ray's path as it interacts with a scene

Access to all contextual scene information.

* Rendering
* Audio/physics simulation
* collision detection
* AI/pathfinding

We'd like to bring this together with rasterization.  Hybrid rendering.
# Hybrid rendering
 1.  Rasterize g-buffer
 2.  Ray tracing pass
 3.  Light composition

```objc
// Create render pass

MTLRenderPassDescriptor* gbufferPass = [MTLRenderPassDescriptor new];
gbufferPass.depthAttachment.texture = gbuffer.depthTexture;
gbufferPass.depthAttachment.storeAction = MTLStoreActionStore;

gbufferPass.colorAttachments[0].texture = gbuffer.normalTexture;
gbufferPass.colorAttachments[0].storeAction = MTLStoreActionStore;
```

 ```objc
 // Create render pass

id< MTLRenderCommandEncoder > renderEncoder =
             [commandBuffer renderCommandEncoderWithDescriptor:gbufferPass];

encodeRenderScene( scene, renderEncoder );

[renderEncoder endEncoding];
```

Now let's encode our raytracing pass.  Dispatch via compute

```objc
// Dispatch ray tracing via compute

id< MTLComputeCommandEncoder > compEncoder = [commandBuffer computeCommandEncoder];

[compEncoder setTexture:gbuffer.depthTexture atIndex:0];
[compEncoder setTexture:gbuffer.normalTexture atIndex:1];

[compEncoder setTexture:outReflectionMap atIndex:2];

[compEncoder setComputePipelineState:raytraceReflectionKernel];

encode2dDispatch( width, height, compEncoder );

[compEncoder endEncoding];
```

After this pass is encoded, we can continue to encode more work such as light accumulation or submit teh command buffer.

Since we encoded our work in two passes, we save our intermediate passes to system memory.  This works, but on apple silicon in iOS devices, there's an opportunity to make this better.

Hardware utilizes tile memory on apple gpus.  At the end of the pass this is flushed to system memory and must be reloaded into the next pass.  But the roundtrip to memory is expensive.  Ideally we avoid this.

Added the ability to dispatch raytracing from inside render pipelines.  This allows mixing render/compute, leveraging tile memory.  Reduces memory use.

[[Enhance your app with Metal ray tracing]]
[[Modern rendering with Metal]]

## Technique enhancements
### Shadows
Challenges for rasterization
* No context of scene
* Render the scene from every light's perspective.
* Render from the main camera's perspective
* Convert from the light's coordinate.
* Determine if we're in light or shadow for each light source.

First, we ahve to render from each light, processing the scene multiple times.

Second, shadowmaps have a predetermined reoslution, meaning shadows are usbject for aliasing
No information for pixels that don't fit into the image

### Ray-traced shadows

Simply trace rays
In the case an object is blocking the pass, we exclude that contribution from the light.  Produces a natural shadow corresponding to occluding object.
No longer limited to info stored in depth map.

We start by rendering from the main camera.  Next, take the acceleration structure from the depth map.

Calculate pix position and trace array in the light's direction.  Determine if lit or in shadow based on intersection.  This process produces a shadow texture.

How to code?

```cpp
// Calculate shadow ray from G-Buffer

float3 p = calculatePosition( depth_texture, thread_id );
ray shadowRay( p, lightDirection, 0.01f, 1.0f );

// Trace for any intersections

intersector< triangle_data, instancing > shadowIntersector;
shadowIntersector.accept_any_intersection( true );

auto shadowIntersection = shadowIntersector.intersect( shadowRay, accel_structure );

// Point is in light if no intersections are found

if ( intersection.type == intersection_type::none ) {
   // Point is illuminated by this light
}
```

Point is in light if no intersections are found

With raytracing, determining shadows becomes a natural technique.  We just trace a ray.  No need to have intermediate depth maps.

### Ambient occlusion
Mute ambient light
Darken crevices

Screen space ambient occlusion (rastering)
Sample nearby objects
Calculate attenuation factor
Create texture

Relying on screenspace information is missing information for nonvisible occluders.

With raytracing, we can rely on real geometric data of the scene.  The idea is for every pixel we generate random rays in a hemisphere and search for intersections against object.  If intersections are found...

 Metal shader for ambient occlusion.
 
 ```cpp
 // Generate ray in hemisphere

ray ray = cosineWeightedRay( thread_id );

ray.max_distance = 0.5f;

// Trace nearby intersections

intersector< triangle_data, instancing > i;

auto intersection = i.intersect( ray, accel_structure );

if ( intersection.type != intersection_type::none ) {
   // Point is obscured by nearby geometry
}
```

Comparison.  SSAO has issues when the geometry is perpendicular to the camera.  e.g. underneath (when viewed from the side).

Raytraced version correctly finds intersections on the floor.  

Also issues with geometry that's offscreen.  Hybrid rendering provides a significant quality improvement, bringing the technique.

### Reflections
Challenges
* Limited resolution
* Requires filtering
* Limited by screen-space

Reflection probes requires strategically placing cameras to capture color information.  Cube maps are captured from different locations.  This is essentially a rendering in 6 directions from the point.  Calculate relative to the probe and sample the cube map.  For realistic results, many probes need to be scattered.  As dynamic objects move across, shaders need more cube maps.

Cube maps need to be pre-filtered to accurately represent irradiance.

Screen-space reflection avoids some problems by basing on pixels on the framebuffer.

Use normals to march outward and check depthmap for nearby objects.
Sample from fraembuffer.  

Suffers from screenspace limitations discussed earlier.  Notice that only part of the surface can get an accurate reflection.  Some parts of the scene is missing.  The lower portion is missing information because it's facing away from the camera.

Raymarching can also get expensive.  

### Raytraced reflections

Reflect the point from a normal.  Trace a ray towards a reflected object.  Provide an accelerated structure.

Calculate view vector from camera to each point, trace ray in direction.  Shade the intersection directly in raytraced kernel.

How to code?  Calculate

```cpp
// Calculate shadow ray from G-Buffer

float3 p = calculatePosition( depth_texture, thread_id );
float3 reflectedDir = reflect( p - cameraPosition, normal );

ray reflectedRay( p, reflectedDir, 0.01f, FLT_MAX );

// Trace for any intersections

intersector< triangle_data, instancing > refIntersector;
auto intersection = refIntersector.intersect( reflectedRay, accel_structure );

// Shade depending on intersection

if ( intersection.type != intersection_type::none ) {
   // Reflected ray hit an object: perform shading
}
else {
   // No intersection: draw skybox
}
```

Comparison.  Additional detail etc.  Shadows, etc.

Those can be achieved by tracing multiple rays.  Because they rely on perfect information, raytraced reflections are free from screen-space artifacts.

[[Explore bindless rendering in Metal]]

For reflections, need to shade points directly int he compute kernel.  Some of these require access to vertex data in the compute kernel directly.  

For these cases, need to make sure GPU h as access to data it needs.  Achieved with bindless binding model, represented as argument buffer.

Check out [[Explore bindless rendering in Metal]]

## Hybrid rendering
* Natural algorithms
* More accurate
* Remove passes
* Saved memory and bandwidth


# Ray tracing  tools
This year, we introduced raytracing support in metal debugger.

In this version of the demo, raytraced shadows have been implemented.  But some shadows are missing.  How can tools help debug?

I have labeled my compute command encoder as "Raytrace shadows"  Consider labeling your ... ?

List of all objects with our current kernel dispatch.  

Acceleration structure.  Now can open and view?  Familiar tools to move around.  Option while scrolling zooms in and out.

Bounding volume traversals - heatmap showing how many nodes a single ray will need to traverse.  Darker colors mean more nodes need to be traversed and lower intersection depth.

Can colorcode based on intersection structures, geometries, instances, etc.  

We need to change the flags.  

Intersector will show us the exact visualization of our scene when using the shadow mask.

[[Discover Metal debugging, profiling, and asset creation tools]]

# Wrap up
* Why ray trace
* Hybrid rendering
* Ray tracing tools

