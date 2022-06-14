# Scaling concept
M1 - up to 8 GPU cores, up to 68gb/s
M1 pro - up to 16gpu cores, up to 200gb/s
m1 max - up to 32 gpu cores, up to 400gb/s
m1 ultra - up to 64 gpu cores, up to 800gb/s

Optimizing for M1 is a great starting point
Great scaling for many applications by default

GPU workload scalability is the increased performance with more GPU cores.
Linear speedup is considered ideal
Minimize GPU gaps
Optimize for GPU limiters

Your goal ist oget as close as possible to linear scaling.  Tools and techniques to identify bottlenecks.

# Maximize scaling
Striving for ideal scaling
* dientify bottlenecks and understand scaling
* often memory or comptuationally bound
* Shift some load to leverage memory
* Bottlenecks can shift with scale
* Use MPS where possible
## Minimize GPU gaps
### Improve work distribution
Identify small workloads.  
Reduce kernel synchronization.

Compute model: grids and threadgroups
Threadgroups distributed to GPU cores
Threadgroups can use threadgroup memory, local to GPU core
Threadgroup consists of SIMD groups (waves/warps)
SIMD group contains 32 threads on Apple GPU
Maximum 1024 threads per threadgroup
CAn share up to 32kb of threadgroup memory (per threadgroup??)

If so... on a 8-core M1, it would be 256kb total?
Or on a 64 core M1 ultra, it would be 2M?
This contradicts other documentation I have seen so it requires further research...

make sure you have enough threadgroups to distribute
Check if there are enough threads to dispatch
1k-2k concurrent rhreads (per gpu core) is considered good occupancy.  ex
|                                 | m1     | m1 pro  | m1 max  | m1 ultra |
| ------------------------------- | ------ | ------- | ------- | -------- |
| GPU cores                       | 8      | 16      | 32      | 64       |
| lowest recommended thread count (TO SATURATE) | 8k-16k | 16k-32k | 32k-64k | 64k-128k         |

Avoid unnecessarily large threadgroups
make TGs small
use smallest multiple of SIMD width (..that maps well to workload)
Large threadgroups (512-1024) limit load-balance opportunities

 This year, on GPU captures, in the righthand panel, we show you the "Max Theoretical Occupancy" which shows how good your shader could be.  Lower than that indicates there might not be enough threads.  Or other causes.
[[Metal Compute on MacBook Pro]]

### Eliminate GPU timeline gaps
GPU timeline gaps never lead to ideal scaling
GPU is mostly underutilized when it's idle
watch for GPU idle gaps

e.g. duty cycle is 50% because stalled on CPU.
GPU is used 33% of the time, latency decreases 33%
Doubling the GPU cores again starts giving diminishing returns
so scaling is not ideal

In the previous example, it was possible to remove CPU-GPU synchronization altogether.  But not always the case, due to application nature.  Other approaches you can take to reduce time.
* Use MTLSharedEvent to signal CPU
	* Lower CPU scheduling overhead.  MTLSharedEvent has lower events tahn `waitUntilCompleted:`?
* Pipeline work in advance
	* If subsequent jobs can be explicitly encoded - do it from the same queue.  By doing so the GPU will not become drained and always have work
	* Otherwise consider using multiple queues
* GPU-driven encoding
	* In some cases an algorithm can do work on GPU.  Use indirect command buffer, can encode directly on the GPU avoiding any need for synchronization.
	* [[Modern rendering with Metal]]
* Concurrent dispatches
	* Compute kernel sycnhorinization has some overhead
	* dispatch has to ramp up and drain
	* when threadgroups finish, might not be enough work anymore
	*  Overlap work when possible


### atomics considerations
Global atomics scalability
Coherent across the whole GPU
Atomics throughput **does not scale with GPU cores**
	leads to **more** contention!!

Suppose we try to sum.  Maybe we use a single atomic in main memory?  Not ideal.  Too much atomic pressure.  This effectively serializes each write.

SIMD-group isntruction (simd_prefix_exclusive_sum, simd_min, many more)
threadgroup atomics (fulfilled by threadgroup memory)

so each group uses a simd thing.
Then each group can reduce to threadgroup memory.
Then finally each group goes to main memory.

maximize your atomics scaling
leverage
* Locality as much as possible
* SIMD-group operations
* threadgroup memory atomics
minimize global memory contention
## Optimize for GPU limiters


### Dispatch grid and data layout tuning
Understand memory access, temoprarily and spatially
Experiment with different
* data layouts
* grid layouts

when compute kernel is dispatched, common to have a 2d-like patternw ith square threadgroups.  This is nto great for data locality.  

instead of spanning row, it's localized into stripes.  With this new memory layout, a threadgroup will be able to utilize most of the data in the cacheline.

can also change how dispatch to better match layout.  Here the access pattern is aligned with the data layout.

* Experiment to find the best fit
* make tradeoffs if necessary
* Every workload is unique
### Blender cycles case study
* reduce memory span access
* maximize memory locality

* default material sorting to reduce thread divergence
	* sort by material type
	* good for thread divergence, bad for memory divergence
* Bucketize data by memory range before sorting
* greatly reduces data divergence

Indexes for material ttype are packed close together.  simd will use index to load corresponding data in original materials buffer.  however, simd will read across the whole buffer.  Pressure on MMU.

So we partition.  Only do data access inside the partition.  In this way we don't let threadgroups read all over.

Top limiter reduced by around 20%.  GPU bandwidth icnreased significantly.

Multiple-levle sorting improved overall performance between 10-30% depending on the scene
your workload may have other ways to improve data locality
keep optimizing top performance limiter

* max theoretical occupancy (new)
* MMU limiter
* MMU utilization
* MMU TLB miss rate
# wrap up
m1 architecture is very scalable
bottlenecks can shift as target device scales
use our tools to identiyf bottlenecks
you need to experiment

 