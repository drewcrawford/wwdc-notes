#metal 

# Advanced Redering Techniques

## Deferred
2-pass approach
Geometry pass: render into gbuffer with normals, albedo, roughness, etc.
Lighting pass: renders the light volumes and builds up the final scene in an accumulation texture

In the geometry pass, we write out depth, + gbuffer (normal,albedo,roughness)
In the lighting pass, we read back the textures, and then we draw the lit scene

Basically we merge geometry and lighting intoa  single pass, where teh G-buffer is kept resident in tiled memory

0.  Make shader changes such as using `[[color(x)]]`
1.  Set load/store actions to `doncare`
2.  Use `.memoryless`


## Tiled Deferred
This tries to solve performance issues with large amounts of lights by introducing a light culling stage.

We cull the lights per-tile and generate a light list for each tile.

Key idea here is we figure out which geometries a tile will hit based on the depth buffer in some way I don't fully grasp
Then look at which lights interact with that geometry.

`.tileFunction` ?  Evidently this is  a property of `MTLTileRenderPipelineDescriptor`.  Not many people seem to use this, it has somsething to do with rendering into or out of tile memory.  Evidently this is specifically used for tile culling, e.g. `tileCullDesc`, not for the normal render pass descriptor, `rpd`.

We set `.threadgroupMemoryLength`

When we go to render,

1.  Do geometry against normal render pass descriptor `rpd`
2.  Bind to tile shader, `dispatchThreadsPerTile`
3.  Now threadgroup memory holds the light list, which we can use in the lighting call.


Tile shader here is `knernel void`, and it has arguments declared like `threadgroup uint32_t &active_light_list [[threadgroup(0)]]`.  

Then fragment shader is `fragment float4 Shade(VertexInputs stage_in [[stage_in]], ... threadgroup uint32_t &active_light_list [[threadgroup(0)]])`



## Tiled and Clustered Forward

Note that we can use the "same persistent threadgroup memory" for an additional pass, e.g. a forward pass.


I don't really understand "Clustered forward" rendering.  Something about using 3d lights instead of 2d lights.

## Visibilty Buffer
So prior techniques depend a lot on tile memory.  How to deal with paltforms taht don't have tile memory?

Reduce data.  Instead of writing gbuffer, only emit barycentrics and aa primitive ID and reconstruct in the final shader.

New attributes to retrieve this data in the fragment shader itself, `[[barycentric_coord]]` and `[[fragment_id]]`

## Demo


# GPU-Driven Pipelines

Stuff you usually do in a render loop
* Frustrum culling
* Occlusionn culling
* LOD selection


To do frustrum/occlusion, you might use the previous frame's data.

Also frustrum culling, is embarassingly parallel.  

1.  Compute pass for frustrum occluders that encodes draws
2.  ...
3.  Frustrum, LOD selection, occlusion culling, encode draws.  Note that occluder data here is no longer dependent on prior frame's data.  So it's more accurate, etc.
4.  Render pass

Need a way to encode draw commands on the GPU.  Indirect command buffers.
Also need scene data.  Argument buffers.

## Argument buffers
Make scene data available on GPU
Describe complex data structures

They envision an argument buffer for meshes, materials, models.  The top-level scene objects.

Each argument buffer is represented by a structure.  It seems the `struct` represents 1 element.

Scene is the top-level `device Mesh *meshes; device Material *materials; ...` struct




## Command buffers
Encode draw on GPU

`device  CommandArgs &comd_args [[buffer(1)]]` <- ICB parameter in the shader

Note the indirect command buffer structure

a command contains
1.  PSO
2.  Vertex buffer
3.  Fragment buffer
4.  draw call args

```cpp
render_command cmd(cmd_args.cmd_buffer, draw_id); //get slot
cmd.set_render_pipeline_state(...);
cmd.set_vertex_buffer();
cmd.set_fragment_buffer(...);
cmd.draw_indexed_primitives(...);
```

To get an icb in swift
`MTLIndirectCommandBufferDescriptor`
`.commandTypes = [.draw]`

`.dispatchThreadgroups`

Likely we `optimizeCommandsInBuffer()` because the order of the buffer may not be optimal?  Seems we do this after `dispatchThreadgroups()`

`.executeCommandsInBuffer()`

Note that ICB is sparse, because some threads will decline to write due to culling etc.  Those slots in the ICB are empty.  If you send this up to the GPU, it will end up executing empty commands, which is not efficient.

Ideally we want to tightly pack the commands.  For taht, we have indirect ... range?

## Indirect range?

1.  Pass in the length of the indirect range buffer.  this is `device atomic_uint *range* [[buffer(2)]]`.
2.  `atomic_fetch_add_explicit(...)` to get the length


In this way we pack the commands and update the range.

Swift side?

First we create a `rangeBuffer`.  We set the buffer to our encoder, then schedule the pass.  So basically the solution I use for `FixedAtomicCVector`.

## Encoding compute dispatches

* Allows GU to build compute dispatches
* Compute ICBs can be reused or modified
* Both render and compute can now be driven on the GPU.
* Build more flexible GPU-driven pipelines.
* 


# Simpler GPU families
Replacing Metal Feature Set queries
* 4 families, focus on cross-platform commonality
* Hierarchical feature instances within each family
* Separate version queries
* Few device queries for optional features

* Apple
* Mac
* Common
* iOSMac - primarily intended for catalyst I think

Argument buffers -> supported by all families
Render/compute indirect command buffers -> common 2 and later

[[Delivering optimized metal apps and games]]


