# What are hangs
https://www.youtube.com/watch?v=YUhMRnEABfc

Laggy, slow, stuck.  Not words you want to use.

## The main runloop
A loop your application's MT enters to run event handlers in response to incoming events, primarily user interactions.  When the user interacts, runloop receives the event, processes, and updates the UI if required.  Happens in 1 turn of the runloop on the MT.

This process repeats for each user input.

If MT takes a lont time, there's a delay.

Events are buffered and cannot be handled by the MT during a hang.  If I interact with an app, that isn't handled until MT first terminates.  This compounds hangs one on top of another.

A delay of over 1s will *always* look like a hang.  A shorter delay can be perceived as one.  By eliminating hangs, your app will be snappy, quick, and responsive.


# What causes hangs
Excessive work on the main thread.  

* MT itself is busy doing work
* MT is blocked by another thread or system resource.

Causes.

## Proactively doing work
More than what's necessary to update UI.  e.g., recipe view only displays images for visible images.

## Performing irrelevant work
MT services blocks on main dispatch queue.  But can service blocks on other queue (dispatch_sync).  When you do this, all pending blocks on the other queue, have to execute prior to the new one.

ONly a fraction of time was spent doing work *meant* for the MT.

Similarly, if a block is dispatched *from* other queues onto MT, that block has to execute on MT.  This holds even if the block is enqueued via dispatch_async.

## Suboptimal API
Many ways to  accomplish a task.  Read API docs, use the best one for the task at hand.

Inefficiently round corners via `UIGraphicsBeginImageContext`.  This operation is CPU-intensive, uses memory, and takes a long time.  The wrong hardware for the job.  Instead of using the CPU, use the GPU.  e.g. `imageView.layer.cornerRadius` etc.

## synchronous APIs
Do not use on the MT if the API does a lot of work or has the potential to block for a long period of time.  Apart from delays, these add an additional point of failure.

One case is if the MT makes synchronous requests to the network.  On slow connections, this may take longer.  No guarantee on how long this can take.  

## File I/O

Most commonly used and contended system resources.  Avoid IO on the main thread.

Datastores which do not support concurrency are especially problematic.  If MT attempts to read from one while writes are already occurring, that read is pushed out until all writers complete.

Another cause for hangs is synchronization.  By definition, sync primitives ccan block execution.  Important to limit and be cautious of syncing from the MT.  Thread it synchronizes with can take a long time to release a lock.

* `@synchronized`
* `.lock()`
* `.sync()`
* `.wait()`
* `os_unfair_lock_lock`
* `pthread_mutex_lock`
* `pthread_rwlock_wrlock`
* `pthread_rwlock_rdlock`

**Specifically, be aware of semaphore use, as they do not propagate priority and can lenghten a hang due to preemption**

Common issue, make async API act synchronous by blocking.  Avoid this on the main thread.

## Getting an unvarying value
State of system resources, cpu memory storage etc., play a large part in when hangs occur.  HW or device conditions mean real-world scenarios will be significantly different when testing on-desk.  Important to do waht you can to defend against these cases by having robust tests and using old hardware as a benchmark.

Too much work being done on the MT.  

# How to diagnose hangs
Now that you know common causes, let's talk about helpful tools to monitor and triage hangs.

To triage a hang,
* Time profiler
* system trace
[[System Trace in Depth - 16]]

Use time profiler and system trace to find hangs demo.

[[What's new in MetricKit]] To collect call trees for hangs in the field.  Prioritize based on common issues.

Important to baseline metrics.  Xcode organizer does this per-app version.
[[Diagnose power and performance regressions in your app]]
[[Improving Battery Life and Performance - 19]]
# How to eliminate hangs
Each strategy can address multiple cases.  In order to know which fix is best, look at side effects and tradeoffs.

## Reduce work on the main thread
* Optimize work on the main thread
* Move work off main thread in a non-blocking manner.

### caches
Great way to quickly access frequently-used assets or previously-queried values.  They're an in-memory store, but can be persisted to disk across multiple invocations.

Formatted assets are great candidates for caching since it's expensive to create them per-time.

By caching these in an NSCache, the overhead of generating assets is replaced with a quick memory read.  This eliminates the hang.

Important to have an accurate cache invalidation mechanism to strike a balance between scaled data and constantly invalidating the cache.  Should happen asynchronously on a secondary dispatch queue.

### Observers

Allow your app to react to changes in a value or state without doing expensive on-demand computation.  Any class can post notifications.  To find notifications, check its api documentation.

Once notification comes in, observer is invoked.  e.g., updating a cached vlaue.  To keep MT responsive, updates should be async.  

### move off

Move work off the MT.  First, determine what this work should be?  In general, important tasks providing critical info should remain on the MT.  VCs etc have to stay on.

However, computation can be offloaded to another thread with a completion handler to perform the actual update on the MT.  This pattern is useful when computation is known to take a long time.

### asynchronus API

Frees the main thread.  By using async NSURL, apps will be responsive.  Often indicated by the word `asynchronously` or `completion` in the method name.

GCD is a powerful multithreading mechanism which you can leverage when there aren't async apis, or the code you want to use is your own.

Prewarm computation.  Task will start executing while the MT is free for other work.  When results are needed, it can dispatch-sync onto the prefetch queue to wait for the task to complete.

[[Modernizing Grand Central Dispatch Usage]]

## Understanding tradeoff
* Caches: Memory invalidation, stale values
* Notifications: coalescing, filters.  Reduce chattiness.
* Async: Priority.  First check if it is crucial for a UI update, as the OS deprioritizes async work.  
* GCD: Changing order at which tasks in your code execute.  Keep in midn what tasks have to be ordered on others to ensure your app does not break.

# When eliminating hangs
* Use apple frameworks and APIs
* Perform improvements iteratively
* Be a good neighbor

# Wrap up
* Set baselines with xcode organizer.
* Identify hang antipatterns
* Triage with instruments and metrickit
* eliminate hangs
* 



