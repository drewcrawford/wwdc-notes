#gcd
GCD is designed to help you scale your application code across various kinds of hardware.

Much of the speed increase in our hardware is about getting smarter, not necessarily faster.  If you "go off core" during an operation, efficiency is reduced.

# Topics
* Parallelism and concurrency
* Using GCD for concurrency
* Unified Queue Identity
* Finding problem spots

# Parallelism and concurrency
* parallelism - simultaneous execution of closely related computations
* concurrency - composition of independency executed tasks

parallelism requires multiple cores.  concurrency is something you can do even on a single-core system, it's about interposing the different tasks.

## parallelism
Split apps into image tiles.

Consider taking advantage of system frameworks
* accelerate - built-in support for parallel execution of image algorithms
* metal/coreimage - take advantage of gpu
* core animation - ?

### `concurrentPerform`
 * express explicit parallelism with `DispatchQueue.concurrentPerform`
 * Parallel for-loop – calling thread participates in the computation
 * More efficient than many asyncs to a ocncurrent queue
 * `DispatchQueue.concurrentPerform(1000) { i in /* iteration i */ }`

Swift automatically chooses the correct context.  
This year, we have a similar thing going on in objc.

```objc
dispatch_apply(DISPATCH_APPLY_AUTO, 1000, ^(size_t i) { /* iteration i */ })
```
Deploys back to macOS 10.9, iOS 7.

#### How to choose iteration count?
Number of cores?  Problem is, let's say we need to do UI rendering as well.  Now the load balancer has to move 1 block over.  This creates a bubble.

"say, 1k".  Large enough so the load balancer has flexibility to move things around.
Be sure to balance the overhead of the load balancer vs the useful work that each iteration does.

Remember, not every CPU is available all the time.
In addition, not every worker thread will make equal progress.

## concurrency
Break app into independent subsystems
* user interface
* networking
* database

What happens when the user clicks a button?

1.  UX renders button highlight
2.  Database decides to refresh the articles
3.  Command to networking subsystem
4.  Meanwhile, user touches the app
5.  OS can immediately switch to UI thread, doesn't need to wait for DB complete
6.  This is the advantage of moving work off the main thread.
7.  Then CPU can switch back to MT.

[[System Trace in Depth - 16]]

### context switches in depth
OS can choose a new thread at any time
* a higher priority thread needs the CPU
* A thread finishes its current work
* Waiting to require a resource
* Waiting for an async request to complete

### excessive context switching
Too much of a good thing.  Repeatedly bouncing between contexts can become expensive, CPU runs less efficiently.

There may be others ahead in line for CPU access.  

causes
* repeatedly waiting for exclusive access to contended resources
* repeatedly switching between independant operations
* Repeatedly bouncing an operation between threads

#### exclusive access to contested resources
Lock contention.
Visualization in instruments.
Staircase pattern.  Each thread runs for a short time and then gives up.

What's going on here is the (UX) thread executes, releases the lock, and some other thread will get it.  The UX continues execution though.  It tries to acquire the lock again, it fails, then we have to context switch to a thread that can make progress.

Alternatively-unfair lock.When lock is released, it isn't reserved by the other thread.  So UX can get it again, and we stay on CPU.  This might make it difficult for second thread to get a chance at the lock, but it reduces the context switches.


|                               | Unfair             | Fair                                              |
|-------------------------------|--------------------|---------------------------------------------------|
| available types               | `os_unfair_lock`   | `pthread_mutex_t`, `NSLock`, `DispatchQueue.sync` |
| Contended lock re-acquisition | can steal the lock | context switches to next waiter                   |
| Subject to waiter starvation  | Yes                | No                                                |

Often the unfair lock works best for objects, properties, global state that may be taken/released many many times.

##### lock ownership

Ownership helps resolve priority inversion
High priority waiter
Low priority owner

Which primitives have this power and which ones don't?

| Single Owner              | No Owner                      | Multiple Owners           |
|---------------------------|-------------------------------|---------------------------|
| Serial queues             | `dispatch_semaphore`          | Private concurrent queues |
| `DispatchWorkItem.wait`   | `dispatch_group`              | `pthread_rwlock`          |
| `os_unfair_lock`          | `pthread_cond`, `NSCondition` |                           |
| `pthread_mutex`, `NSLock` | Queue suspension              |                           |

We support this only for "single owner" column.
No owner makes this impossible.
Multiple Owners – maybe can implement this someday?

Consider whether or not your usecase involves threads of different priorities interacting.  If so, consider a primitive with ownership, ensuring that your UI thread doesn't get delayed  by waiting on a lower priority bg thread.

##### optimizing lock contention
Inefficient behaviors are often emergent properties
Visualize your app's behavior with instruments
Use the right lock for the job


# other talks
[[Simplifying iPhone App Development with Grand Central Dispatch - 10]]
[[Asynchronous Design Patterns with Blocks, GCD, and XPC - 12]]
[[Power, Performance, and Diagnostics: What's new in GCD and XPC - 14]]
[[Building Responsive and Efficient Apps with GCD - 15]]
[[Concurrent Programming with GCD in Swift 3 - 16]]

# serial dispatch queues
Fundamental GCD primitive
mutual exclusion
FIFO oredered
Concurrent atomic enqueue
Single dequeuer

```swift
let queue = DispatchQueue(label: "com.example.queue")
queue.async { /* 1 */ }
queue.async { /* 2 */ }
queue.sync { /* 3 */ }
```

In practice, 	`queue.sync` here will enqueue a placeholder into the queue, the thread can wait until that placeholder pops.  Now an automatic worker thread will come along to execute prior work items, then the ownership of the queue will transfer to your thread, to do the sync bit.

# dispatch source
Event monitoring primitive
* event handler executes on target queue
* Invalidation pattern with explicit cancellation
* Initial setup followed by activate
* Note that there's a general OS pattern "like this" where we deliver events to you on a target queue.

```swift
let source = DispatchSource.makeReadSource(fileDescriptor: fd, queue: queue)
source.setEventHandler { read(fd) }
source.setCancelHandler { close(fd)}
source.activate()
```

# target queue hierarchy
* Serial queues and sources can form a tree.
* Shared single mutual exclusion context
* Independent individual queue order Q1 vs Q2
	* However only 1 work item executes at a time in total
	* May be heterogeneously interleaving Q1 and Q2 items

```swift
let Q1 = DispatchQueue(label: "Q1", target: EQ)
let Q2 = DispatchQueue(label: "Q2", target: EQ)
```


# QoS
* abstract notion of priority
* Provides explicit classification of your work
* Affects various execution properties

[[Power, Performance, and Diagnostics: What's new in GCD and XPC - 14]]

In our tree, any node (source or queue) might have QoS attached to it.  E.g., let's say we have a source for `userInteractive`.  
Can give a QoS to the final EQ.  The effect of this is nothing in the queue executes below this level, for example we can say `>= utility`.  Now if S1 fires, we use `utility` if there's no higher QoS.


Source firing is just like "an async that happens from the kernel".  For asyncs from userspace, qos is usually determined from the thread calling `queue.async`.  So if that enqueuing thread is `userInitiated`, we expect that qos.

Priority inversion when EQ has items like `[utility,userInitiated,userInteractive]`.  This is resolved by bringing up the EQ worker thread with the highest priority of any item in its queue.  So the EQ is now executing with `userInteractive` priority.

# Granularity of Concurrency
Let's talk about Networking.  We have dispatch_source in the kernel.
In any networking subsystem we have many such sources for different connections.

Each source goes to a differnet queue.  So at this point we've asked the system to potentially execute n things at once here.  If all become active at once, system will create n threads for you.  This may be what you wanted, but it is quite common for these event handlers to be small, and then enqueue data into a datastrctures.  This can lead to a situation where you have tons of context switching, because each one is a small amount of work.

Can apply a single mutual exclusion context by giving a EQ at the bottom.  In this way, we are only doing 1 at a time.

## Avoiding unbounded concurrency
Repeatedly switching between independent operations

Many queues becoming active at once
* Independent per-client sources
* Independent per-object queues

Many work items submitted to global concurrent queue at a time
* if workitems block, more threads get created
* may lead to thread explosion

[[Building Responsive and Efficient Apps with GCD - 15]]

### one queue per subsystem
Maybe we have 1 serial queue for database, networking, user interface?

How about one queue *hierarchy* for each subsystem?  Then can taget the network, database queue, etc.

The main thing is to have a *fixed number of serial queue hierarchies*.  Maybe one for fast work, one for slower work items, etc, so the first one can stay responsive.

Also think about granularity.  Want to use fairly large work items to move between subsystems so the context switch is long enough to reach efficiency.  

Once you're inside the subsystem, it may make sense to subdivide into smaller work items, which will not introduce a context switch because you're inside the subsystem.

## summary
* organize queues and sources into serial queue hierarchies
* Use a fixed number of serial queue hierarchies
* size your workitems appropriately

# Unified Queue Identity
When you create EQ, you might dispatch async to it.  Earlier, we would get a worker thread 'anonymously'.  In this release we changed that, and we create an identity tied to your queue, in the kernel.  The object has the right qos.  That is used to ask for a thread.

The thread request may not be fulfilled for some time, e.g. background thread, system says not now.

I think the idea here is we have some variable in the kernel which we can store the current qos of the queue.  So this allows us to do priority inversions every time we `sync` or `async` into that queue, we are potentially updating that kernel variable to a new qos.

This means that when we have 

```swift
let S1 = DispatchSource.makeReadSource(fileDescriptor: fd, queue: EQ)
S1.setEventHandler { ... }
S1.activate() //S1 now has qos of EQ
```

Now let's consider a higher-qos S2
```swift
let S2 = DispatchSource.makeReadSource(fileDescriptor: fd, queue: EQ)
S2.setEventHandler(qos: .UserInteractive) { ... }
S2.activate()
```
What we're trying to solve here is "repeatedly bouncing an operation between threads".

In previous releases, we request a thread "anonymously", so we don't know what the priority is.  So we get this anonymous thread, then we notice the source targets a queue, so we bring the queue over to the anonymous thread.

S2 fires, we bring up an anonymous thread.  Context switch.  Now we notice this needs to run on EQ, so our high-priority thread is now waiting.  The context switch did not produce any useful work.  Now we context-switch back to the first thread, and then context switch back to S2.

With unified identity, thread and queue are basically the same object.  So we don't context-switch anymore.  When the event fires, we  know where it will execute.  We just mark the thread.  At the first possible time, we notice the thread is marked as "have pending events".  Then we dequeue the events, and can enqueue a new handler.

Why did we explain this?  So you can understand how to best take advantage.  The runtime uses every possible hint to optimize behavior.  

# Modernizing existing code
## No dispatch object mutation after activation
Set the properties of inactive objects before activation
* source handlers
* target queues

```swift
let mySource = DispatchSource.makeReadSource(fileDescriptor: fd, queue: myQueue)

mySource.setEventHandler(qos: .userInteractive) { ... }
mySource.setCancelHandler { close(fd) }

mySource.activate() // no changes after here!
```

At activate-time, we take a snapshot, and we make future decisions based on that snapshot.

If you make changes, priority and ownership snapshots can become stale
* defeats priority inversion avoidance
* defeats direct handoff optimization
* defeats event delivery optimization

System frameworks may create sources on your behalf
* XPC connections are like sources

## Protecting the target queue hierarchy
Build your queue hierarchy bottom (EQ) to top (source).

You can opt into "static queue hierarchy".  If you're using Swift 3, you're already doing that.  For your existing codebase, 

```objc
Q1 = dispatch_queue_create("Q1", DISPATCH_QUEUE_SERIAL)
dispatch_set_target_queue(Q1,EQ)

//vs
Q1 = dispatch_queue_create_with_target("Q1", DISPATCH_QUEUE_SERIAL, EQ)
```

# Finding problem spots in an existing app
## demo
available in an upcoming seed

# Summary
* Not going off-core is ever more important
* Size your owrk appropriately
* Choose good granularity of concurrency
* Modernize your GCD usage
* Use tools to find problem spots



