#metal 

Most powerful chips ever!
16-core GPU => 200gb/s, 32gb
32-core GPU => 400gb/s, 64gb

# Recap of Metal Compute
* Unified graphics and compute
* Efficient multithreading
* Precompiled or online shaders

1.  Command queue.  Application queues work at some point
2.  Command buffer.  Transient.  May imposed around CPU/GPU.  But you'll want to ensure you ahve enough work to keep the GPU busy
3.  Command encoder.  Graphics, blit, etc.
4.  Kernel dispatch.  
5.  End encoder
6.  Commit
7.  Completion handler, OR
	1.  waitUntilComplete

Multiple command buffers can be encoded simultaneously.  Can split across threads.  Can reserve place in the queue.

# New MBP best practices
## GPU memory
UMA.  CPU and GPU access the same memory.
Avoid GPU copies and use shared resources.  This drastically reduces the memory bandwidth.

For a contention, say CPU write then GPU read, a multibuffer is necessary.  CPU writes to buffer N, GPU reads N-1.

Query MTLDevice for the recommended maximum working set size.  (single command encoder).  `defaultDevice.recommendedMaxWorkingSetSize`.  Control how much memory you use.

Allocating more than is possible and residency is managed by Metal.
|               | System memory | GPU working set size |
|---------------|---------------|----------------------|
| M1 pro/m1 max | 32GB          | 21GB                 |
| M1 Max        | 64GB          | 48GB                 |


## Submitting work
Look to batch encoders together into each command buffer before calling commit. 

If you wait for GPU to find out what to dispatch next, bubbles will form.  To hide this, consider using multiple CPU threads working on multiple pieces of work, and keep GPU busy.  

Where possible, increase threads in kernel dispatches.  

With smaller thread counts, use the concurrent dispatch model
`cmdBuffer.makeComputecommandEncoder(dispatchType: .concurrent)`.  

## Resource selection
Buffers vs textures
* Separate L1 caches on GPU
* If your app is only using buffers, there are performance benefits to moving some resources to textures.  Better utilization of high-performance caches, reduce traffic, increase performance.
* Texture data can be twiddled.  Texels ordered more optimally for a random-access pattern.  Transparent to the kernel so it doesn't add complexity.
* Also lossless compression of a texture.  Further reduce texture bandwidth.  Also transparent.  Compressed by default if private, but shared/managed can be explicitly compressed.
* MTLTextureUsage must be exclusively `shaderRead` or `renderTarget`
* If your texture data is actual image data, consider lossy compression such as BC or ASTC.

# Kernel optimizations

* Memory indexing
* Global atomics
* Occupancy
* Kernel profiling

[[Optimizing Metal Performance for Apple Silicon Macs]]

## Memory indexing
* Index memory use signed int types over unsigned

This appears to be because signed overflow is undefined, which we can exploit to do a vectorized load.

## Global atomics
Minimize the use of atomic operations in your kernels
Consider thread-group atomic based techniques instead
## Kernel profiling
Utilization -> kernel used x% of the capability
Limiter -> GPU is bottlenecked by ALU for x% of the time

Why are these different?  Limiter can be thought of as the efficiency of  work type.  Time spent doing actual work + time spent on internal stores or inefficiency.  Best case, equal.  In practice there's a difference.

Occupancy.  

## Occupancy
Measure of how many threads are active on GPU relative to max.  When low, important to understand why to determine if this is expected.

Causes of low occupancy
* Limited resources (??)
* Exhausing thread or thread-group memory
* High register pressure

Solutions
* Decrease the amount of shared memory used
* Determine maximum thread count at PSO creation time `maxThreadsPerthreadgroup` or in MSL kernel source.  `[[max_total_threads_per_threadgroup]]`  Smallest multiple of thread execution width that works for your algorithm.

### Register pressure
"Spilled bytes" in instruments?  "Temporary registers".  Occupancy is reduced to free up these registers.  
Need to reduce by "up to the block size" to translate into an increase in occupancy.

solutions
* Use 16-bit short and half over int and float
	* conversions are usually free
* reduce stack data
* `constant` address space.  Drastically reduces number of general-purpose registers used.
* Avoid dynamic indexing for stack or constant data. 

[[Optimize metal performance for apple silicon macs]]

# Wrap up
* Recap of metal compute
* New MBP best practices
* Kernel optimizations


