#metal 

# Introduction

CPU/GPU share System Memory
GPU has on-chip Tile Memory.  **No** dedicated video memory!

DRAM bandwidth is a common bottleneck.
GPU is a TDBR.

Two phases:
* tiling (geometry)
* rendering (pixels)


## Tiling

1.  Split viewport into a list of tiles
2.  Shade all vertices
3.  Bin trasnformed primitives into tiles.

## Rendering

For each tile
1.  Load action
2.  Rasterize
3.  Shade all visible pixels
4.  Store action 

The more GPU cores we have, the more tiles we can shade at the same time.

## GPU configuration
GPUs have mutliple cores

Each core
* Shader core (ALU)
* Texture core (TPU)
* Pixel backend
* Dedicated tile memory

Threadgroup and imageblock data is stored in Tile Memory
All metal resources are stored in Device Memory

Note the memory hierarchy here:

* Shader core has dedicated L1
* Texture unit has dedicated L1
* Tile memory is per-core
* All cores share a "GPU last level cache"
* Finally we get to system memory

[[Harness apple GPUs with metal]]
[[Delivering optimized metal apps and games]]

## Understanding the complexity

Rendering a frame requires multiple passes
on multiple GPU cores
each processing multiple tasks
on different ahrdware units
which have different throughputs

FLOPS -> alu throughput
mbps -> tpu throughput

## GPU performance counters

answer questions like.. does the gpu have enough work?
too much work?
perf bottlenecks?
what takes the longest?

# Metal System Trace

Good overview.

# Metal debugger

Deep perf investigation
Unaffected by thermals or dynamic system changes

# What exactly do the values mean?

## Performance limiters

GPU execute work in parallel.  Math, memory, rasterization.
Limiter counters measure the activity of multiple GPU subsystem.  Find work executed, find stalls.  Limiters point you to the slowest path.

"Game Performance" template in Instruments.
Recording options => Metal Application => Performance LImiters under "GPU counter set"
Also enable "Shader Timeline (iOS)"

### Arithmetic

ALU - processes arithmetic operations.  Bitwise ops (and,or,etc)
relational - add,subtract
optimized for floating-point
optimized for coherent execution (SIMD)

Rates – A13

16-bit fp - 2.0x
32-bit add/sub - 2.0x
32-bit fp - 1.0x
32-bit integer - 0.5x
complex (i.e. log2, exp2, etc) - 0.5x (best case).  Some operations, e.g. sqrt, are slower

EAch thread in the SIMD executes the same instruction.  SIMD units have 32 threads.  Single program counter across all 32.  So we want all threads to execute the same instruction.


Note that a SIMD group (32 threads) is the smallest grouping of work for GPU.  Scheduling e.g. 2 threads will spin up all 32.

What can I do if ALU limiter is high?  **Celebrate**.  Maybe this is ok.

Otherwise,

* Replace complex calculations with approximations or LUT
* Replace fp32 with fp16.  Avoid implicit conversions, avoid fp32 inputs (e.g. in buffers)
* Use `-ffast-math`


### Texture read/write

Metal textures
backed by device memory
Read by **Texture Unit**
* Has dedicated L1
* Support for multiple filtering modes
* Support for multiple compression modes

Written by the **Pixel Backend**.

Note that read/write is different hardware block!

#### TPU 
Reads texture data
Render pass executes `MTLLoadActionLoad` for attachment
read/sample from a shader

Optimized for gather operations
Optimized for "regular pixel formats"

Sampling rates (A13)

Conventional 32-bit formats - 1.0x
RGB11B10Float, RGB9e5Float, RGBA16Float - 1.0x
YUV (GBGR422 and BGRG422) 64-bit formats - 0.5x
128-bit formats (e.g. RGBA32Float) - 0.25x

Texture filtering rates (A13)

1d,2d,cube,nearest/linear - 1.0x
1d,2d,cube mipmap linear - 0.5x
3d nearest/linear - 0.5x
3d nearest/linear - 0.25x
2d 8xanisotropy - 0.125x
shadow (pcf) - 0.5x

Compressed formats.  Apple GPUS support
* block-compressed (PVRTC or ASTC)
* Lossless compression of conventional format

256x256 cube map (no mips)
ASTC_8x8_HDR - 96
RGBA16Float - 3072

So how to fix high texture sample limiter?

* use mipmaps if minification is likely occuring
* Change filtering options
	* Use bilinear instead of trilinear
	* Use lower anisotropic sample count
* consider using smaller pixel sizes
* Leverage texture compression
	* block-compression (ASTC) for assets
	* Lossless compression for textures generated at runtime

#### Texture *write*.
Pixel backend writes texture data, e.g. for `MTLStoreActionStore` or explicit write to texture from a shader.

Optimized for coherent writes
* write-combined cache
* Avoid divergent writes (different tiles, different array indices, etc...)

Texture write rates (A13)

8,16,32-bit formats - 1.0x
64-bit formats - 0.5x
128-bit formats - 0.25x

Note that pixel backend and TPU are different hardware blocks.  So they have different throughputs (a13):

* TPU texture read - 1.0x
* PB texture write - 0.5x

How to fix high texture write limiter?

Consider using smaller pixel sizes
If using MSAA
* Reduce sample count
* Reduce number of small triangles

Optimize for coherent writes
* Improve spatial/temporal locality


### Tile memory load/store

tile memory, what is it?

Unified set of high-performance memory
Stores threadgroup and image-block data

Accessed when
* reading or writing pixel data from imageblock
* Reading or writing from threadgroup memory
* Reading or writing to render pass color attachments
* Enabling blending on a render pipline

What if I see a high value?

* Reduce the number of threadgroup atomics
	* Consider threadgroup parallel reductions
	* Consider SIMD/quadgroup operations
* align threadgroup memory allocations/accesses to 16 bytes
* Reorder memory access patterns
	* Neighboring threads access to neighboring elements
	* Remove accesses to the same location by multiple threads

### Buffer read and write

Metal buffers, how do they work?

* Backed by device memory
* Accessed by Shader Core
	* Dedicated L1
	* Support for different address spaces
* Address spaces for buffer data
	* Device (read-write, not cached).  Use for data indexed per-fragment or per-vertex
	* Constant (read-only, cached, pre-fetched).  use for data utilized by many vertices or fragments.

What can I do if I see a high value?
* Pack data more tightly
	* Use smaller types (i.e. `packed_half3` for position)
* Vectorize loads/stores
	* Use SIMD types (i.e. `float4`)
* Avoid "device atomics"
* Avoid register spills
	* Remove dynamic indexing into thread-scoped arrays
* Use textures to balance the workload.  Note that ALU/TPU have *different caches*

### GPU last level cache

Shared across *all GPU cores*

* caches texture and buffer data
* Stores device atomics
* Optimized for spatial and temporal locality

Memory instruction peak rates (A13)

* Tile memory local access -> 1.0x
* GPU last level cache global access -> 0.5x
* Tile memory threadgroup atomic -> 0.25x
* GPU last level cache device atomic -> 0.125x

Note that we favor tile memory over GPU LLC, and watch out for atomics!

What do I do if I see a high value?

If texture or buffer limters show a high value
* Optimize those first
* Reduce size of working sets

If shaders use *device atomics*
* Refactor code to use *threadgroup atomics*

Access memory with better spatial/temporal locality

### Fragment input interpolation

What is it?

Fragment Input is inteprolated during the rendering stage by the *Shader Core*

The SC has a dedicated *Fragment Input Interpolator*

* Fixed function
* Full precision (FP32)

What do I do if i see a high value?

Remove vertex attributes passed to the Fragment Shader

# Learn more about GPU profiling
Check developer docs for these articles

* Optimizing Performance with the GPU counters instrument
* Reducing shader bottlenecks
* Measuring the GPU's use of Memory Bandwidth
* Finding your app's GPU shader occupancy

# Memory bandwidth

Device memory is backed by *System memory*

Stores texture data, buffer data, tiled vertex buffer.  Cached by gpu last level cache (LLC)

*Membory Bandwidth GPU Counter* measures memory transfers between GPU And system memory.  GPU accesses *System Memory* when buffers or textures are accessed.

Note that *System Memory* includes a *System Level Cache* (SLC).  So you may see transfer bursts at a higher rate than true DRAM throughput.

What can I do if I see a high value?

If texture or buffer limiters show a high value
 * optimize those first
 * Reduce size of working sets

Only load data needed by current render pass
Only store data needed by future render passes

Leverage texture compression
* Block-compression (ASTC) for assets
* Lossless compression for textures generated at runtime

# Occupancy

GPUs have a maximum number of threads that can run at the same time.
* Occupancy measures how many threads are running
* Latency is the time it takes to complete a task.

GPUs hide latency by switching between available threads.  They create new threads when

* there are enough internal resources to do so
* Threre are commands scheduled to run

Note that if a task has high latency, for example because we are transferring memory, GPU will switch between threads to avoid a stall.

`computePipeline.maxTotalThreadsPerThreadgroup` - the max number of threads in a threadgroup that can be dispatched using the pileine

`computePipeline.threadExecutionWidth` the number of threads that are executed simultaneously by the GPU
`computePipeline.staticThreadgroupMemoryLength` the length, in bytes, of the threadgroup memory that is statically allocated.

## Occupancy Counter
Measure the percentage of total thread capacity being used by the GPU.
The total percentage is the sum of compute, vertex, fragment occupancy.

Note that neither high, nor low occupancy indicate a problem.
* Low vertex occupancy is ok if there is high fragment occupancy
* low occupancy is fine if the GPU resources are fully utilized.

Overlapping work in compute, vertex, etc may increase occupancy.


What do i do if I see a low value?

* Correlate occupancy measurements with data from other counters or tools
* When overall occupancy is low
	* Shaders have exhausted some internal resources
	* Threads finish executing faster than the GPU can create new ones
	* Your app is rendering to a small area, or idpsatching very small compute grids, such that the GPU runs out of threads to create

# Hidden Surface Removal

What is it?
Minimize overdraw by keepign track of front-most visible layer for each pixel.

Pixel-perfect
Submission-order-independent for opaque meshes

Pixels are processed in 2 stages
1.  HSR
2.  Fragment processing

## Measure efficiency

Use GPU counters to measure
* Number of pixels reasterized
* Number of FS invocations
* Number of pixels stored
* Pre-Z test fails

Overdraw is the ratio between FS Invocations and Pixels Stored

What to do?
* Minimize full-screen passes
* Minimize blending

But also

Draw meshes sorted by visiblity state
1.  Opaque
2.  Alpha test / discard / depth feedback
3.  Translucent

Avoid interleaving opaque and non-opaque meshes
Avoid interleaving opaque meshes with different color attachment write masks

# Demo
Reorganized counters into groups.
You can create your own groups.


Note that there's both an "encoder" tab and a "draw" tab.  Switch to the "draw" tab and filter by the encoder you identified as the problem earlier.

[[Delivering optimized metal apps and games]] - optimize for GPU option

# Further items
[[gain insights into your metal app with xcode 12]]
[[Modern rendering with Metal]]