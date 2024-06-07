Learn how the new profiling tools in Xcode 15 can help you achieve the best Metal performance on Apple family 9 GPUs. Discover how to use shader cost graphs, performance heat maps, and shader execution history tools to profile and optimize your Metal code. Find out how to use new GPU counters to optimize GPU occupancy and ray-tracing performance.

* New shader core architecture
* State of the art performance analysis tools
[[Explore GPU advancements in M3 and A17 Pro]]


# New profiling tools

Analyze performance.  Two approaches
* identify expensive shaders
* Locate costly objects or pixels

New performance analysis tools
## Shader cost graph

## Performance heat maps
Shader thread information and performance metrics
Fragment position
Compute thread ID

Shader execution cost
Thread divergence
Overdraw.  Render opaque objects first!
Instruction count
Draw ID (each gpu command a different color).

Heat map tab.
Click plus button to show more heatmaps.

## Shader execution history
Click on a pixel in the heatmap to select underlying simd group.  This will reveal shader execution history for the simd group below the heatmaps.
First time I can see exactly how simd groups are executed by apple gpus!

Under my most expensive function, loop with 12 iterations.  Too many lights *too many lights*

SIMD group execution
Thread state changes
Call stack
Loop detection
# Occupancy profiling

[[Explore GPU advancements in M3 and A17 Pro]]

family 9 gpus: m3, a17 pro.  GPU in both chips have varous components.

ALU pipelines, memory pipelines.
On-chip memory

On-chip memory has L1 cache.

## performance and occupancy

Suppose you have ALU, buffer read, use result.  This may require going all the way out to device memory, long latency.  During this time, SIMD group can't execute other operations, which causes ALU pipelines to go unused.

To mitigate this, shader core can execute instructions from different group.  

SIMD groups that ar e currently running is called occupancy.  To achieve optimal performance, increase occupancy utnil everybody is as busy as possible

## occupancy management

Regitser, threadgruop, tile, stack, are assigned dynamically from L1 cache.  Backed by last cache and device memory.
On-chip memory usage increases with thread occupancy.  May use more memory than available on chip.  Spills to next cache level.
Dynamic occupancy adjustment for better performance.

New performance counters in xcode 15.  Identify causes of low occupancy.
Workflow to triage occupancy.

Determine metal workload occupancy.
Encoder section, counter section.

## utilization and limiter
* work: items processed in a hardware block, such as ALU instructions
* stall: available items held off by downstream block (ex waiting from cache or memory)

utilization = 100 x (work in sample period) / ( peak processing rate * sample period)

limiter = 100 * ((work + stalls) in sample period) / (peak processing rate * sample period)

## Triaging low occupancy

limiter?  Do less of that.
No limiter?  Something else
* Increase occupancy until it is not a bottleneck
* Inspect the shader launch limiter counter (ex workload size too small)

Small counter value for launch limiter indicates that the shader cores are getting starved because the workload is too small.

high value: either enough threads are getting launched, or there's backpressure.


counter value is too high:
* too much threadgroup memory?
	* See how much threadgroup memory is dispatched in right panel.  2kb is low.
* Occupancy Manager Target counter
	* Lower than 100%.  Indicates that occupancy manager is engaged by GPU to keep various shader data memory types on chip and avoid spilling to memory.
		* L1 eviction rate counter.  How much register/tile/stack memory are able to stay on chip or get spilled to next cache.  High spikes: L1 cache being thrashed.
		* L1 load/store.  High Imageblock L1 store bandwidth.  And high load bandwidth.  Residency by types.  Here we want to use smaller pixel formats, to reduce L1 imageblock size.
		* L1 eviction low?  Last level cache stalls / MMU stalls.
			* Last level cache counter.  Includes utilization time, and stall time.  If limiter is higher than utilization, it indicates getting stalled a lot due to cache thrashing.  Reduce buffer size, improve spatial/temporal locality.
			* MMU limiter: device buffer accesses are causing TLB misses.  Reduce incoherent memory access to buffers can help reduce stalls.




# Ray tracing profiling

With new ray tracing hw accelerator, render realistic life-like scenes in realtime.  How metal debugger optimizes performance?

Used raytracing to render some nice reflections.  Fast, but I'm curious to see how to make it faster.

* Ray tracing counters
* Acceleration structure viewer

## Ray tracing counters
* rays active.  We optimize ray occupancy to ensure max performance.

Assuming we're not starved by number of rays, let's start with occupancy manager target.  Low?  L1 rate.  L1 high? L1 residency/bandwidth.  If high? optimize RT scratch usage
then last level cache,  mmu, etc.

Can also tell you what rays are working on.  ex, 75% are performing instance transform.  

Try to maximize opaque triangle test.

## Acceleration structure viewer

Find a  dispatch where I use acceleration structure.  Encoder in timeline.  Select the dispatch.

See instance traversals.  

I should concatenate instances together into a single structure.  Could be a symptom of a problem in my asset pipeline.  

Continue to follow best practices

[[Your guide to Metal ray tracing]]
[[Maximize your Metal ray tracing performance]]
[[Learn performance best practices for Metal shaders]]

# Wrap up
* shader cost graph
* Performance heat maps
* Shader execution history
* Occupancy profiling
* Ray tracing profiling
