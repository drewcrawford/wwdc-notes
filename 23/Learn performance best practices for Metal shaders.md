Discover how you can improve Metal shader performance using some of the latest advancements in Apple GPUs. Learn to reduce a shader's execution time by configuring function constants, and investigate ways to increase compiler optimization with function groups. Find out how to save run time by improving the shader's execution and ability to use resources in parallel. Explore the Apple family 9 GPU features and take advantage of hardware acceleration for ray tracing.

Apple family 9 gpu: m3, a17
guidances for all apple gpu generations

# Reduce shader execution time

function constants to specialize efficiently
function groups to optimize indirect function calls

## function constants

eg., ubershaders, if we have divergent controlflow based on uniform parameters.

shader has to account for several possibilities, and read from additional buffers.  May affect an app's performance.

Specialize shader at compiletime instead of runtime.  Preprocessor macros.

Specializing at build time results in large libraries.  Even if you compile them offline AOT, each variant adds up which can significantly increase the size of your metal library.  Also increase compile time because each shader variant has to be compiled starting from source.

Function constants can reduce compiletime and library size.  One time from source, then we specialize.

Use function constants to initialize program scope variables.  Disparate codepaths instead of metal buffers.

Metal can fold these as constant booleans as it compiles the variant.  As well as other optimizations, such as eliminating unreachable codepaths.  Remove unused control flow.  By specializing shaders with constants, don't need to query multiple parameters from buffers.  Reduces shader runtime by simplifying codepaths.

[[Optimize GPU renderers with Metal]]

## function groups

indirect function calls -> without invoking the name.  e.g. function pointers, or visible function tables.
May be static or dynamic linked.  May obstruct optimization across function calls.

For statically linked functions, can use metal function groups to optimize with indirect function calls.  

When you know that the function pointers can only point to one of a specific group, can use groups attribute.  ex one group for lighting, one group for materials, etc.  `[[function_groups]]("lighting")`.

Specify in `MTLLinkedFunctions`.  Only helps for functions you statically link.  Binary library doesn't benefit.

[[Get to know Metal function pointers]]
[[Discover compilation workflows in Metal]]

## wrapup
* function constants
* function groups
# Improve resource utilization
* higher thread occupancy can improve latency hiding
* thread occupancy depends on resource availability
	* registers
	* memory
* better resource utilization leads to better thread occupancy
[[Explore GPU advancements in M3 and A17 Pro]]
[[Discover new Metal profiling tools for M3 and A17 Pro]]

## choose right address space

* support different access patterns
* Memory region to allocate from
* Address space selection impacts performance

constant:
* read-only
* constant data across all threads
* fixed size and is read many times

device:
* read/write buffers
* variable data across all threads
* variable size

[[Optimize metal performance for apple silicon macs]]

threadgroup:
* read/write memory objects
* threads in a threadgroup can share data
* software managed cache of device/constant buffers

Profile shaders to guide address space use
* consider using device/constant buffers instead of software managed cache in threadgroup memory
* we now have flexible on-chip memory.  We use same cache hierarchy as buffers now.  If your working size fits in cache, both buffer and threadgroup access may have similar performance characteristics.
* profile and evaluate using metal debugger in xcode

## choose right data type for ALU ops

16-bit datatypes can reduce register/memory footprint
* use half precision and short type as much as possible
* conversions are free
* bfloat type
	* accelerates ML applications
	* wide range of values at a lower precision
	* supported since Metal 3.1
* Can lead to better thread occupancy
* better energy efficiency

Shader with a mix of float, half, in ttypes
ALU pipelines execute in parallel
Choose the right datatype for ALU operations

# apply ray tracing best practices
Define your scene geometry.  Build an AX structure to allow efficient intersection.
Intersection is performed from a GPU function that creates a ray. makes an intersector object to perform intersection.

Use custom intersection function only when necessary
reduce ray payload
minimize the number of intersection tags
prefer intersector over query

## use custom fn only when necessary
ex, when we traverse alpha, maybe we need to ignore a transparent pixel in the alpha layer.

opaque intersectors are the fastest path.
Avoid duplicating intersection functions
Enables optimal grouping
Simplify intersection function tables

intersection function.  Intersection SIMDgroup

* avoid instructions that reduce parallelism of the intersection function
	* side effect instructions such as memory writes
	* indirect function calls
	* If needed, place them as late as possible
RT scratch is used to store some data.


## reduce ray payload
We have a per-ray payload.  The larger it is, the more impact on RT performance.  Avoid user payload, prefer intersection result.

Specialize per intersect() call.
Use packed datatypes (ex packed_float3)
Remove any unused or unneeded fields

we don't need position since it can be ray.origin + result.distance * ray.direction

and can pack RGB to uint.

## minimize number of tags
* tags contribute to ray tracing scratch usage
* object matrices have to be stored for each ray

tags must match between
* the intersector, and
* the intersection functions it calls
## prefer intersector over query
intersector does not use custom functions.  So we have to wait.

if you choose query
* intersection function sorting is lost
* more ray tracing scratch is required
Intersector aligns with HW implementation, prefer over query.

If you do need query
* change properties if needed, re-use the query object.  Enables re-use of RT scratch.
* Do not overlap intersection queries if you need multiple.ex don't do 1,2,1.  Continue with 1, then switch to 2.

## recap
* use custom fn only when necessary
* reduce payload
* minimize tags
* prefer intersector over query
[[Your guide to Metal ray tracing]]
[[Discover new Metal profiling tools for M3 and A17 Pro]]

# wrap up
* reduce execution time
* Improve resource utilization
* apply ray tracing best practices

