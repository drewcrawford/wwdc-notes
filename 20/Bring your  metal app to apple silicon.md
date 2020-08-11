Powerful TBDR architecture used on iphone, appletv, etc.

#metal

Supports a unified metal featureset that combines macOS and iOS.

[[optimize metal performance for apple silicon macs]]

# adapting your app for the apple gpu
# common issues coming from intel-based macs
# metal techniques for consistent rendering
|                   | apple silicon mac                            | intel-based mac         |
|-------------------|----------------------------------------------|-------------------------|
| gpu               | apple gpu                                    | intel, nvidia, amd gpus |
| gpu architecture  | tile-based deferred renderer                 | immediate mode renderer |
| metal API support | metal GPU family mac 2 metal GPU maily apple | metal GPU family mac 2  |

## Immediate renderer
Looks like this has a kinda fixed pipeline to me, it's unclear if it's really fixed or just shown that way on this diagram.

Importantly, vertexes may be non-adjacent, so the hardware needs to cache large parts of the buffer, (e.g. the whole buffer), not just a subset.

## Tile mode

The first step is creating a database of geometry in memory.  That I'm going to refer to as tile vertex buffer.
Since we have the whole geometry for an entire tile, we can rasterize up-front.  

Via this method, we know something about faces, and we don't shade any fragment that will be occluded by another one.  We can do this with just tile size buffer on the chip rather than entire depth buffer.  

It does not matter if a triangle is is full-screen, or how the draw calls is organized.

If we're not going to use the depth buffer, we can use memoryless data for the depth buffer, so that works out well too.

Can do blending and alpha testing without ever having to load the color buffer in memory.  We donly need tiled color buffer on the chip.

## Recap
Tiling - geometry
Rendering - pixels 

Tile processing allows blending to occur in-register.
No reason to re-fetch color depth or stencil buffers.

# OpenGL and OpenCL

Deprecated, not removed.

# Recap
All mac gpu and compute APIs are supported.
Follow best practices for the tile-based deferred renderer architecture
For better performance, make use of the new APIs.

# Common issues coming from intel-based macs

## Metal feature detection
Query features directly.  Don't use `#target`, etc.

## Load and store actions
On apple gpus, directly control state of on-chip api memory
[[optimize metal performance for apple silicon macs]]


Note if a dontCare action is chosen, apple gpus will not move the texture from system memory to tile memory

## Position invariance
Results of the same vertex position calculation can vary slightly
Performance (default) vs accuracy tradeoff
Note that if you want invariant vertexes, you must enable this explicitly

Note that this likely required for depth buffer `equal`, because the position will need to be computed to the same value in 2 cases (passes?)

## Threadgroup memory synchronization

simd size is 32 on apple gpus
If there is only 1 simd group per threadgroup, no need to synchronize threadgroup

You might want to rewrite your shaders for simdsize 32 as threadgroup sycnhronization is more expensive than simd

## Sampling depth and stencil attachments
A texture used as an attachment cannot be sampled in the same render pass.

Do not use texture or memory barriers.  Those are very expensive. 

If your app requires sampling the current attachments, create a second copy for sampling.

## Metal techniques for consistent rendering
| workarounds                            | macOS catalina sdk | new macos sdk             |
|----------------------------------------|--------------------|---------------------------|
| dontcare load action                   | Use load instead   | use dontcare as requested |
| position invariance                    | forced             | off by default            |
| sampling depth and stencil attachments | snapshot           | no snapshot               |

[[optimize metal performance for apple silicon macs]]

