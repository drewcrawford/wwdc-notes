#metal 

# apple TBDR GPUs
very power efficient
unified memory architecture
* cpu/gpu share system mory
* gpu has on-chip tile memory

Note that there is no "video memory"

"tiling" - vertex
"rendering" - fragment

## Tiling
* split viewport into a list of tiles
* shade all vertices
* Bin transformed primitives into tiles

GPUs don't have a large pool of dedicate dmemory, so where does the post-transform data go?  Into the "tiled vertex buffer".

## Tiled vertex buffer
Stores tiling phase output
* Post-transorm vertex data
* other internal data

Mostly opaque
* affected by MTLStorageModeMemoryless

Causes "Partial Render" if full
* GPU will split the render pass to flush

## Rendering
Where the TBDR architecture shines the most.

Previously, GPU split into a list of tiles. Now we render each tile.
For each tile:

* Load action
* Rasterize
* Shade all visible pixels
* Store action

Mostly opaque, but the application controls load/store actions.


## Vertex fragment stage overlap


## Hidden surface removal

On-chip depth buffer.  Allows the GPU to minimize overdraw by keeping track fo the frontmost visible layer for each pixel.

HSR is independent of the draw order, so if you draw back to front, we still won't do overdraw.

The implication of this is we first run all vertex shaders, before running any fragment shaders.  With the vertex shader we rasterize the primitive into the depth buffer.

On a scene with non-opaque geometry, overdraw depends on HSR efficiency.

So you want to draw meshes sorted by visibility state.

1.  Opaque
2.  Alpha test / discard / depth feedback
3.  Translucent

Avoid interleaving opaque and non-opaquue methods
Avoid interleaving opaque messages with different color attachment write masks.

Render all opaque geometry first.  

Note that ther eis no dedicated blending unit, we do this in tile memory


## Programmable blending
Allows fragment shaders to access pixel data directly from tile memory.  This means we avoid load/storing into system memory, which is less efficient.

## memoryless render targets
?
## efficient MSAA
`MTLSToreactionResolve` will save memory bandwidth.
`MTLStorageModeMemoryless` will also save memory footprint.

# modern apple GPUs

A11 and newer - major GPU redesign

Now have an on-chip imageblock, as well as a new programmable stage called Tile Compute.

## Imageblock

2D data structure accessible from shaders.  Width, height, depth.
Accessed from fragment functions or kernel functions.

Imageblocks - load and store data using a single operation, rather than per-pixel.  This is more efficient.

## Tile shading
Mid-render compute

Dispatchesa re interleaved with draws
Executed in API submission order
Dispatches barrier against earlier/later draws

### Enhanced msaa

You can now access the data inside the 'memoryless' buffers and resolve in your render pass

One issue: some people render a lot of translucent particle effects, which complicates msaa.  Instead, you can do an explicit "resolve msaa" pass after your opaque geometry and before the translucent geometry.

Resolve is fully programmable now.

## explicit control of tile memory
* tile shaders
* imageblocks
* persistent threadgroup memory

## GPU-driven rendering
* argument buffers
* indirect comamnd buffers

Classically, the CPU decides what to render based on occlusion data in the previous frame.

### Argument buffers
Make scene data available on GPU
describe complex datastructures.

### Indirect command buffer
Issue draw commands.

[[Modern rendering with Metal]]

# next steps

[[optimize metal apps and games with GPU counters]]
[[gain insights into your metal app with xcode 12]]
