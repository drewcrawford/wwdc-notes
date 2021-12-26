#applesilicon 

Games are highly demanding.  Hundreds/thousands of CPU jobs per frame.  16ms or less.

# Apple silicon CPU
This year, we introduced M1pro, m1max.  Efficiently tackle very demanding workloads.
CPU, GPU, neural engine, many more.
High-bandwidth low-latency unified memory.  

On M1, cores of 2 types.  P core, E core.  Transparent to developers.  Don't care where thread ones.

Cores arranged in clusters according to type.  Each cluster has last-level cache.  

Energy-efficiency and workload-adaptive.  Each cluster may be independently activated.

Note the availability of P-cores is not guaranteed.  System reserves the right to make them unavailable in some thermal scenarios.

XNU
POSIX and Mach objects
NSObjects, GCD, app, etc.

# CPU efficiency fundamentals
Types of overhead
* Core wake-up cost.  Core otherwise idle.
* Scheduling cost.  Overhead from OS scheduler
* Synchronization latency (semaphores, etc.)

Instruments trace.  Too fine granularity.  18us.  We can see two issues here.

1.  Synchronization primitives are used at an extremely high frequency, that introduces overhead.
2.  Active work is extremely brief.  Only lasts between 40-20us.  This duration is so small that it is barely shorter than the time it takes to wake a core.

Mostly waiting for a core to wakeup.  

## Right granularity
Group tiny jobs into larger ones
Running a thread introduces a schedluing cost
Tiny jobs will ahve a lower ratio of work over synchronization
CPU will be better utilized
Larger jobs amortize the scheduling cost

## Line up work before leveraging threads
Get most jobs ready at once
Signaling and waiting on threads may lead to performance hits
Threads will be moved on and off core
With nested for-loops, parallelize the *outer* for

## ex trace
We can see sychronization latency here.  Two issues here.  

1.  Actual work is extremely small (again).  
2.  Context switches.  Gaps between scheduled work. 

## Use a job pool
Leverage worker threads with job stealing
Scheduling a thread has a cost for both the kernel and CPU
Much cheaper to start a job in userspace!
Workers should keep grabbing jobs instead of waiting
Wake up workers just as needed
Ensure enough work is lined up to justify.

## Make every cycle count
Wait on explicit signals
Avoid inefficient patterns
* busy-waits monopolize a CPU core for nothing
* `yield` is equivalent to `setpri(0)` on macOS/iOS
* `sleep(0)` is a no-op.

## Scale the thread count properly
Match thread count to CPU core count
To omany threads generate more context-switching
Too few limit parallelization opportunities
Query the CPU layout at launch time

| Parameter identifier       | definition                                          | values on M1          | values on M1          |
|----------------------------|-----------------------------------------------------|-----------------------|-----------------------|
|                            |                                                     | P cores (perflevel_0) | E cores (perflevel_1) |
| hw.nperflevels             | Number of types of general purpose cores in the SoC | 2                     | 2                     |
| hw.perflevel{N}.cpusperl2  | Number of cores (perf level N) sharing an L2 cache  | 4                     | 4                     |
| hw.perflevel{N}.spusperl3  | Number of cores (perf level N) sharing a L3 cache   | N/A                   | N/A                   |
| hw.perflevel{N}.logicalcpu | Number of enabled logic cores (perf level N)        | 4                     | 4                     |

Query data per-core type.  0 is the most performant.
Refer to documentation.
Number of active threads per-CPU core.  

Thread state trace. => amount of thread state changes and their duration.  Context switches view, count of context switches per process.

Useful metric to measure elapsed schedulign efficiency.

## Wrapup
1.  Choose the right job granularity
2.  Line up work before leveraging threads
3.  Use a thread pool with Job Stealing
4.  Make every cycle count
5.  Scale your thread count properly


# APIs in practice
## GCD
a.k.a. libDispatch
General-purpose job manager
Optimized and integrated in the kernel, thinks about thermal pressure, P/E core ratio, etc.
Provides Dispatch Queues
Leverages a shared in-process thread pool.  Depends on the type of queue and job properties.
Shared for entire process.  Within a process, libraries share pools.

`dispatch_async_f`

* Incremental thread count ramp up
* At job start, wakes up another thread if more work is enqueued

`dispatch_apply_f`
* Goes right away
* Allows pushing a single job for parallel problems (parallel for)

[[Modernizing Grand Central Dispatch Usage]]
[[Building Responsive and Efficient Apps with GCD - 15]]

## Custom job managers
### Prioritizing threads
Games typically have hundreds of jobs per frame, with varying importance

Apps can set prioritization with **either**
* A raw CPU priority value
* A QoS class

#### CPU priorities
value telling how important computational throughput is
* Ascending value => more important
* Hints at whether a thread runs on E or P cores
* Doesn't priotize access on other hardware resources

#### QoS classes
Improves OS responsiveness with regards to other processes
Used to prioritize tasks for many system-wide resources.  Network, disk, timer coalescing, etc.

| Class                      | recommended usage                    | Max CPU priority (default) | Min CPU priority |
|----------------------------|--------------------------------------|----------------------------|------------------|
| QOS_CLASS_USER_INTERACTIVE | Per-frame work                       | 47                         | 38               |
| QOS_CLASS_USER_INITIATED   | Asynchronous/cross-frame work        | 37                         | 32               |
| QOS_CLASS_DEFAULT          | Streaming / multiple frames deadline | 31                         | 21               |
| QOS_CLASS_UTILITY          | Background asset download            | 20                         | 5                |
| QOS_CLASS_BACKGROUND       | ⚠️May not run for a very long time    | 4                          | 0                |

priority - fine-tune priority in the same class.
Be careful with bg class, threads using it may not run at all for a very long time.


```c
pthread_attr_init
pthread_attr_set_qos_class
pthread_create
pthread_attr_destroy
```

```
pthread_set_qos_class_np
```

Note that if you set a raw value *instead* of calling the np function, you opt out of qos for that thread.  **Permanent and you cannot opt back in for that thread afterwards.**

## Priority decay
iOS and macOS deal with many processes
Priority decay may be used when system is overloaded.

OS slowly decays thread priorities over time so older threads have a chance to run.
* Critical threads may also be preempted by background activity
* This can be controlled with scheduling policies

Maybe opt out of priority decay?

By default, you get `SCHED_OTHER`
* Default, time sharing policy
* Compatible with QoS classes

`SCHED_RR`
* Fixed priority, better consistency ine xecution latency
* Designed for consistent, periodic, high priority work only
* No QoS

|                 | Thread                        | Scheduling policy | QoS class / priority               |
|-----------------|-------------------------------|-------------------|------------------------------------|
| Per-frame       | Main thread                   | SCHED_OTHER       | QoS user interactive (priority 47) |
| Per-frame       | Render thread                 | SCHED_RR          | 45                                 |
| Per-frame       | workers - high priority       | SCHED_RR          | 39-41                              |
| Per-frame       | Workers - medium/low priority | SCHED_OTHER       | QoS User Interactive (priority 38) |
| Multiple frames | Async workers - high priority | SCHED_OTHER       | QoS user initiated (priority 37)   |
| Multiple frames | Prefetching/streaming         | SCHED_OTHER       | QoS Default (priority 31)          |
| Multiple frames | Other low priority tasks      | SCHED_OTHER       | QoS Utility (priority 20)          |

Never use SCHED_RR for long duration work.  

Note that GCD or system frameworks try to inheret QoS.  This assumes QoS is available on the parent.

## Priority inversion
* happens when high-priority thread waits on low-priority thread
* Stalls the high-priority thread

The system might solve it in some cases by boosting low-priority thread.

| Single owner - handling priority inversion | No owner - can't handle | multiple owner            |
|--------------------------------------------|-------------------------|---------------------------|
| os_unfair_lock                             | dispatch_semaphore      | pthread_rwlock            |
| pthread_mutex_t                            | pthread_cond            | Private concurrent queues |
| NSLock                                     | NSCondition             |                           |
| GCD serial queues                          | GCD queue suspension    |                           |
| DispatchWorkItem.wait                      | dispatch_group          |                           |

## Memory
ObjC frameworks manipulate autoreleased objects
Added to a list so that deallocation happens later
* Scopes limiting the lifetime of auto-released objects
* Help reduce peak memory footprint

Threads shoul dhave at least one AR pool in their entrypoint.
If manipulates without one, will cause a leak.
Multiple AR pools can be nested.  
* Render thread (per frame)
* Worker threads (per job)

Avoid competing writes by multiple threads to the same cache line
"False sharing"
On apple silicon, a cache line is 128 bytes
* Add padding int he memory layout to reduce write conflicts

# wrapup
* apple silicon cpu
* CPU efficiency fundamentals
* APIs in practice
* 