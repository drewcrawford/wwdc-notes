CPU + GPU objects
Allocations vs actual use

# Understand game memory
debug navigator, gauges.  
Current memory use.  Understand what this means?
Memory use is NOT allocations.  (On physical memory.)
Memory use is physical, allocations are virtual.

When your game allocates memory, those allocations do not immediately go to physical memory.  We will reserve some space in virtual memory.  When program uses the allocation, we assign to physical memory.

Allocations are grouped into categories and sparsely occupy virtual address space.
* executable
* libraries
* stack
* heap
* resources
* metal resources

regions.
Under the hood, memory allocations work as pages.  16 KiB on modern apple devices.
Only used pages are on physical memory.  System charges these to your game.

* Dirty
	* Memory that you've written to.  
	* Heap allocations
	* Written framework symbols
	* Accessed metal resources (apple silicon, CPU and GPU share same pool of memory)
* dirty compressed
	* Least recently accessed dirty pages
	* system will compress if not used
	* may be swapped to disk
	* Your game is still charged for uncompressed size.
* clean
	* Mapped files.
	* Read-only frameworks
	* Don't count toward memory footprint.
	* may be resident

Usually most interesting to look at dirty/dirty compressed.  We call these two "memory footprint" and system uses these to enforce the memory limit.

*now you know*

Get available memory.
```objc
#import <os/proc.h>

API_UNAVAILABLE(macos) API_AVAILABLE(ios(13.0), tvos(13.0), watchos(6.0))
size_t os_proc_available_memory(void);
```

Get memory footprint
```c
#if __has_include(<libproc.h>)
#include <libproc.h> // On macOS.
#else
#include <sys/resource.h> // On iOS, iPadOS and tvOS.
int proc_pid_rusage(int pid, int flavor, rusage_info_t *buffer)  
    __OSX_AVAILABLE_STARTING(__MAC_10_9, __IPHONE_7_0);
#endif

rusage_info_current rusage_payload;
     
int ret = proc_pid_rusage(getpid(),
                          RUSAGE_INFO_CURRENT, // I.e., new RUSAGE_INFO_V6 this year.
                          (rusage_info_t *)&rusage_payload);
NSCAssert(ret == 0, @"Could not get rusage: %i.", errno); // Look up in `man errno`.
        
uint64_t footprint       = rusage_payload.ri_phys_footprint;
uint64_t footprint_peak  = rusage_payload.ri_lifetime_max_phys_footprint;
```

We  reviewed
* allocations: on virtual memory address space
* Memory footprint: actual memory use
* Dirty + compressed/swapped pages
* CPU + GPU objects on apple silicon
* Memory limit enforcement
API to query memory use
* current/peak footprint
* available memory
# Profile memory growth
Gauge.  You can get a more detailed look by profiling in instruments.

You may want to profile your launch rather than attaching to an existing one.
New "Game memory" template this year.

Allocations, metal resource events instruments.  VM Tracker.  Virtual memory trace.  Metal application and GPU.

demo etc.

 In instruments, click the record button, or use xctrace
 ```bash
xctrace record --template "Game Memory" \
               --attach ModernRenderer \
               --output ModernRenderer.trace \
               --time-limit 30s
```

```bash
xctrace record --device-name "Seth's iPhone" \
						   --template "Game Memory" \
               --attach ModernRenderer \
               --output ModernRenderer.trace \
               --time-limit 30s
```

In metal games, you may find a lot of things in anonymous VM, in VM:IOAccelerator (metal resources).

IOSurface => Drawables.
Can change display track to be density (allocations per time).  Spikes may be sources of memory growth.

Metal resource events track.  History of resource allocations and deallocations.  Identify the metal resources by their labels which you can specify programmatically through the API.

VMTracker.  Noncompressed dirty and compressed/swap memory.  Dirty size.  "Swapped size" is compressed **or** swap.  

Recap
* start with game memory template
* record, analyze, repeat

[[getting started with instruments]]
[[developing a great profiling experience - 19]]

# analyze memory graph
File to efficiently store a complete snapshot of memory state.  Take a snapshot when an issue occurs, or a pair before/after for comparison.

Ingredients
* your game
* configuration of MallocStackLogging
* memory graph

```bash
# See `man malloc`.
MallocStackLogging=lite # Live allocations only.
MallocStackLogging=1    # All allocation and free history.
```

live => discards deallocated objects.  Most of the time, live is the recommended option.

Click on debug memory graph in debug area, xcode will take a memory snapshot.
left inspector, right inspector, middle.
export memory graph in file menu.

can capture memory graph programmatically
```bash
leaks $PID     --outputGraph foo.memgraph
# or
leaks GameName --outputGraph foo.memgraph
```

What to do with it?
1.  Break down memory use by categories
	2. Use `footprint`.  For game memory, `IOAccelerator` is usually the largest.
	3. Heap allocations `MALLOC_LARGE` etc.  
3. distinguish dirty and compressed
4. check class instances
5. trace object allocation call stack
6. inspect object references

You can tag anonymous memory for easier debugging.

with mmap:

```c
size_t length;

int tag = VM_MAKE_TAG(VM_MEMORY_APPLICATION_SPECIFIC_1); // Check out `man mmap`.
    
void * reservation = mmap(NULL,
                          length,
                          PROT_READ | PROT_WRITE,
                          MAP_ANONYMOUS | MAP_PRIVATE,
                          tag, // Instead of using default `-1`.
                          0);

if (reservation == MAP_FAILED) {
    @throw [[NSError alloc] initWithDomain:NSPOSIXErrorDomain
                                      code:errno
                                  userInfo:nil];    
}

return reservation;
```

with mach_vm_allocate:
```c
size_t page_count;

mach_vm_size_t allocation_size = page_count * PAGE_SIZE;
mach_vm_address_t vm_address;
kern_return_t kr;

kr = mach_vm_allocate(mach_task_self(),
                      &vm_address,
                      allocation_size,
                      VM_FLAGS_ANYWHERE | VM_MAKE_TAG(VM_MEMORY_APPLICATION_SPECIFIC_1));
    
if (kr != KERN_SUCCESS) { // Refer to mach/kern_return.h.
    @throw [[NSError alloc] initWithDomain:NSMachErrorDomain
                                      code:kr
                                  userInfo:nil];
}
    
return vm_address;
```

In `footprint` dirty **includes swap and compressed.**

## Distinguish dirty and compressed
`vmmap --summary`.
This separates dirty and swapped sizes.  Swapped includes original size of compresesd or swapped memory.
Adds these together to determine footprints.  But since swapped isn't used as often, it's a good indicator of what to look for to optimize your game's memory.
Virtual size => allocation size
resident size => **includes clean pages**

Shows heap allocations in separate table.  Grouped by zones.  
DefaultmallocZone, MallocHelperZone => based on size
QuartzCore zone
Check % frag. [[Detect and diagnose memory issues]]

Use vmmap without `--summary` shows reach region line by line.  Just like how virtual address space is.

## Check class instances
`heap --quiet foo.memgraph`
`--quiet` : skips metadata in output
New this year, heap is more intelligent at identifying object types.  uses information from ammloc stack logging to present caller or responsible library.

Breaks down library usage.  Be more informed on how memory is spread out.  Hints which direction to go for getting more information on these allocations.

Sometimes, sorted by class size is heplful `--sortBySize`.   
`--showSizes` (show each object, not summaries)

how to find a single object?
`heap --quiet mfoo.memgraph --addresses "NSConcreteMutableData.*[10M-]"`
we get the address.


## Trace object allocation call stack
`malloc-history --quiet foo.memgraph --callTree 0xaddress --invert`

also with memory debugger.
or `malloc_history --quiet foo.memgraph --callTree "VM_ALLOCATE.*"`
check for anonymous VM usage.  

## Inspect object references
`leaks --quiet foo.memgraph --traceTree 0xaddress`

with xcode 14, we redesigned memory graph view to show incoming and outgoing edges.  Can choose edges to draw in a popover.

Consider using Leaks and memory graph view to find important object reference relationships.

## Recap
* enable mallocstacklogging
* capture a memory graph with xcode or leaks
* find
	* large and troublesome objects
	* Object allocation backtrace and usage

[[Detect and diagnose memory issues]]
[[iOS memory deep dive - 18]]
# Optimize metal resources

1.  Resource isnights
2. resource size
3. resoruce recent usage
4. texture pixel format
5. texture compression
6. resource storage mode
7. resource heap

metal debugger.

Can filter resources from GPU capture.  
Insights column – tells you stuff that might help
allocated size - sort to see largest resources
time since last bound - see what haven't been used recently
set purgeable state to volatile?
* non-volatile
	* Set to volatile if not used frequently
* volatile
	* query and reload when reused.
* empty

## Textures
Can see additional columns in right click.
Many textures can use a 16-bit half-precision format.

Bits per component
number of components
Block compression (read-only textures)

* aSTC, BC, etc
* prepare with textureconverter

On a15+, can do lossy copmerssion on render targets.

[[Discover advances in Metal for A15 Bionic]]
[[Discover Metal debugging, profiling, and asset creation tools]]

## Resource storage mode
If texture is only used by a single pass use **memoryless**.  Temporary like depth, sample, multisample.
If only used by GPU, use *private*
Or else, shared/managed.

Managed is not needed on apple silicon macs, just like on iPhone/iPad.

## Heaps
Multiple resources backed by the same memory allocation
[[Go Bindless with Metal 3]]

need to manually sycnhronize resource accesss


## Recap
* use the checklist to audit metal dresources
* use memory viewer in metal debugger

[[Delivering optimized metal apps and games]]
[[Discover Metal debugging, profiling, and asset creation tools]]

# Wrap up
Memory footprint = dirty + compressed/swapped
Profile game memory growth in instruments
Analyze memory graph with tools in terminal and xcode memory debugger
Optimize metal resources in metal debugger
















