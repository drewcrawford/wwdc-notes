#coreaudio

If you use realtime threads, this talk is for you
Please adopt new API

# Should I create my own realtime threads?

Probably not.
But if you do, measure.

Performance controller helps assign threads to CPUs.

Makes decisions based on running applications and threads, etc.

Now, the performance controller understands the workgroups api.

# Workgroup
* group of threads
* Master thread wakes up periodically
* tells the kernel when it begins work, what the deadline is
* may signal and wait for other threads, potentially in other processes
* tells the kernel when it completes work

It seems like AU threads join the workgroup automatically.  However, your own threads might not?  Need to join these to workgroup

1.  Obtain the device's `os_workgroup_t`
	2.  HAL, AUv2, AUv3
3.  Call `os_workgroup_join()` and `os_workgroup_leave()`

Threads may have differnt deadline.  First, create your own workgroup for you rown deadline.

Then join workgroup threads.  

IN the "mster" thread for the workgroup, on each work cycle, call `os_workgroup_interval_start(workgroupInterval, startTime, deadline, NULL)`
startTime is often `mach_absolute_time()`
It can be in the past (if you know you're late)
dealdine must be > startTime
units are `mach_absolute_time()`.  Don't assume they're ns!
`os_workgroup_interval_finish(workgroupInterval, NULL);`

## Audio units
The vast majority of audio units don't create extra rendering threads, but some do

`AUREndercontextObserver`.  Implement this to return a block that receives a context.  This struct contains an `os_workgroup_t`.

When workgroup changes, `os_workgroup_join` and `leave`.  Be aware that the workgroup may change from one render call to the next.

The system provides the workgroup to the AU.  Host does nothing extra.

**All audio threads should be joined to workgroups**

Non-realtime audio threads have no deadlines, and cannot be joined to workgroups.

Parallel worker threads – `os_workgroup_max_parallel_thread()` will recommend how many threads to use.

In the AU case, you can't call this because you don't have a workgroup yet.  So, first you spin up 1 thread per core, then you use this API, which may be less than the threads you created earlier.

| System audio frameworks             | Frameworks join threads to workgroups, you don't                            |
|-------------------------------------|-----------------------------------------------------------------------------|
| App: parallel to device thread      | Join device's thread to workgroup                                           |
| App: asynchronous to device threads | Create and manage workgroup Join threads to workgroup                       |
| AU: paralle to host rendering       | Implement `AURenderContextObserver` Join thread(s) to workgroup it observes |

