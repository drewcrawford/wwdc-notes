Learn how Dynamic Caching, the next-generation shader core, hardware-accelerated ray tracing, and hardware-accelerated mesh shading of Apple family 9 GPUs can improve the performance of your Metal apps and games.

Apple family 9
a17 pro
m3 

we use gpus.  
we use shaders.

**Massive data parallelism**

Metal shader programs many times over in parallel on different inputs.  

# Next-generation shader core

Demos.

Compute/vertex command processors
rasterizer
GPU last level cache
shader cores.  (paired with texture unit, RT unit).

shader core also contains:
* ALU pipelines (fp32, fp16, integer, complex)
* memory pipelines (texture access, buffer access)
* on-chip memory (register file, threadgroup and tile memory, buffer stack and cache)
* SIMDgroup pool, SIMDgroup scheduler

Dynamic shader core memory
* higher thread occupancy
Flexible on-chip memory
* more efficient data access
High-performance ALUs
* increased parallel execution

Suppose we 
1.  ALU
2. buffer read
3. more alu?

long latency ALU starvation.  Shader core can execute instructions from a different SIMDgroup here.  Reduces ALU starvation.

Number of SIMDgroups running on a core is the 'thread occupancy'.  What dictates this?

ex shader
1.  Ray intersect
2. inspect result
3. different shading function based on material.

More/fewer registers used per line.  In this example, implementation of shadeGlass uses many more registers than the rest of the program.

Prior to apple9, simdgroup cannot begin execution until it gets registers from the register file.  Max register # at any point in the program.  Registers stay allocated for the whole duration.

Dynamic shader core memory: we no longer depend on max simd group.  Now on-chip memory is dynamically allocated and deallocated according to what each part of the program uses.  Much more efficient use of on-chip register file.

Register file is now a cache instead of permanent storage.  More registers can be used than can be stored on-chip.  Threadgroup/tile memory are now a cache too.  Now everything is cached on-chip, we redesigned the on-chip memories into fewer larger caches that serve all memory types.

* most programs don't use all memory types
* more space available to the memory types that are used
benefits:
* higher occupany
* large buffer working sets
* faster function call performance

too much memory:
* we spill to next cache level, or core memory
* we dynamically adjust occupancy to maximize performance
* keeps data on-chip and execution pipelines busy
* all shader memory types can impact occupancy!

If you do need to optimize occupancy further, we have profiling tools.

[[Discover new Metal profiling tools for M3 and A17 Pro]]
[[Learn performance best practices for Metal shaders]]

## ALU pipelines

fp16, fp32, etc.  Apple gpus are highly optimized for fp16.  Use wherever possible.

High throughput
Fewer registers
Smaller memory footprint and bandwidth
Free format conversions

ALU pipelines executei n parallel
up to 2x performance for programs that perform combinations of integer, fp32, and fp16 math
execute instructions from different simdgroups
higher occupany improves alu utilization

But, if we interleave fp16/fp32 instructions, then fp32 can overlap with fp16.

## recap
* dynamically register memory improves occupancy
* Large on-chip cache available to all memory types
* Occupancy changes dynamically to memory usages
* FP16, FP32, integer ops execute in parallel

# Hardware-accelerated ray tracing

[[Your guide to Metal ray tracing]]
[[Enhance your app with Metal ray tracing]]

Intersector object, finds intersection by calling object's intersect method.
To determine the intersection point, we do a few key stages.
* traverse acceleration structure
* Intersection function (maybe provided by app)
* Update closest intersection
* Process repeated until closest found
* Closest returned to calling GPU function for app-specific processing.

New, we hardware-accelerate the intersector.

Does not execute inline with GPU function, thus to faciliate communication, data is read/written to onchip memory, which you can observe with RT search performance counters.

How these are executed.  Typically, not all traverse takes the same amount of time.
Execution divergence, causes each thread in a simd group to wait for the longest traversal before proceeding.

overhead compounds with intersection functions too.  Each (type of) function runs one after another.  Large idle times waiting on other threads.

Hardware accelerated is different.  First, hardware intersector is able to run each traversal independently with fixed function hardware.  Arrays are set to hardware intersector for processing instead of executing inline.  Decreases the time spent traversing, removes overhead of divergence.

On the other hand, the interesection functions are MSL, so they still must be grouped into SIMD groups.  However, because the hardware intersector works independently, free to group together arrays from separate simd groups.  We can re-order to run all same function on same simd group, to minimize divergence.

## best practices
* prefer the `intersector<intersection_tags...>::intersect()` API.  Much better than query API, doesn't have reorder stage
* Create one intersection function per unique logical intersection type
* Minimize size of ray `[[payload]]` type.  Decrease shaders's latency, increase thread occupancy.

[[Learn performance best practices for Metal shaders]]
[[Discover new Metal profiling tools for M3 and A17 Pro]]
[[Maximize your Metal ray tracing performance]]

## recap
hardware acceleration is good
* fixed function traversal
* intersection function reorder stage
Prefer intersect API.
# Hardware-accelerated mesh shading

Two compute-like shaders.
* object shaders
* Mesh shader (meshlet)
* Rasterizer
* fragment shader

Flexible pipeline for GPU-driven geometry
* fine-grained geometry culling
* procedural geometry
* compressed geometry inputs
* porting geometry and tesselation shaders
[[Transform your geometry with metal mesh shaders]]
[[Bring your game to Mac]]

Increased performance
* reduced memory traffic

Indirect command buffers
* encode mesh draw commands on GPU
Increased maximum mesh grid size
* 1,048,575 total threadgroups

## Best practices
* Minimize set of `metal::mesh<...>` template parameters
	* Size of vertex and primitive types (remove unused attributes)
	* maximum vertex and primitive counts.  Keep these small.  Reduces memory traffic, may increase occupancy
* Don't write culled primitives to the output `metal::mesh` object.
	* Completely omit such primitives.
# Wrap up
* Next-generation shader core
* Hardware-accelerated raytracing
* Hardware-accelerated mesh shading


