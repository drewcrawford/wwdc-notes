# Prerequisite talks
[[Meet asyncawait in Swift]]
[[Explore Structured Concurrency in Swift]]
[[Protect mutable state with Swift actors]]

We hope this talk gives you a better mental model of how to reason about Swift concurrency and how it interfaces with existing APIs like GCD.

# Threading model
Highlevle components of sample app
* UX thread
* Db
* Networking

How to structure in GCD?

1.  MT: event gesture
2.  dispatch async to DB serial queue.  Ensure MT is responsive.  Access to DB is protected since serial queue guarantees mutual exclusion.  
3.  While here, we iterate through newsfeeds the user has subscribed to, and for each of them schedule a networking request
4.  As responses come in, URLSession delegate callback.  Concurrent.  In completion handlers, we sync update db with latest request from each feed.  Cache for future use.
5.  Wake up MT to refresh the UI.

This seems reasonable.  Let's take a look at a code

```swift
func deserializeArticles(from data: Data) throws -> [Article] { /* ... */ }
func updateDatabase(with articles: [Article], for feed: Feed) { /* ... */ }

let urlSession = URLSession(configuration: .default, delegate: self, delegateQueue: concurrentQueue)

for feed in feedsToUpdate {
    let dataTask = urlSession.dataTask(with: feed.url) { data, response, error in
        // ...
        guard let data = data else { return }
        do {
            let articles = try deserializeArticles(from: data)
            databaseQueue.sync {
                updateDatabase(with: articles, for: feed)
            }
        } catch { /* ... */ }
    }
    dataTask.resume()
}
```

This code has hidden performance pitfalls.

First, dig into how threads are brought up to handle work.  In GCD, when work is enqueued, system brings up a thread to service the work item.

Since a concurrent queue can handle multiple work items, system will bring up several threads until we've saturated CPU cores.

If the thread blocks, and there's more work to be done, GCD will bring up more threads to drain work items.

Reasons:
1.  Give your process another thread, each core continues to have a thread that executes work at any given time.  Your app has a good continuing level of concurrency
2.  Blocked thread may be waiting on a resource, such as sempahore, befor eit can make further progress.  New thread may be able to help unblock the resource that's being waited on.

On a 2-core device, GCD brings up 2 threads.  As the threads block, more threads are created to continue working on the networking queue.  CPU then has to switch between different threads processing results.

This means that in our news application, we could easily end up with a very large number of threads.  if the user has 100 feeds, each URL data tasks has a cmpletion block on the concurrent queue when the network request completes.  GCD brings up more thraeds, when each callback block.

## Excessive concurrency
* Overcomitting the system with more threads than CPU cores.  
* Thread explosion

[[Modernizing Grand Central Dispatch Usage]]
[[Building Responsive and Efficient Apps with GCD - 15]]

* Performance costs
	* Memory overhead
	* Scheduling overhead

Each blocked thread is holding onto valuable memory and resources while waiting to run again.  Each has stack, kernel structures to track the thread.

Some threads may be holding onto locks, which other threads might need.  This is a large number of resources and memory to be holding onto for threads that are not making progress.

Scheduling overhead.  As new threads are brought up, the CPU needs to perform a full thread context switch to execute the new thread.  As the blocked threads become runnaable again, have to timeshare the threads on the CPU, so they are all able to make forward progress.  This is fine, but when there's thread explosion, having to timeshare hundreds of threads can create excessive context switching.  Scheduling latency outweighs the useful work they would do.  CPU runs less efficiently.

## Now
you see we have 2 threads, no context switches.  All blocked threads go away, and instead we have a lightweight object, known as a continuation.  When threads execute work, they switch between continuations instead of performing a context switch.

Now we only pay the cost of a function call.  

So we want to create only as many threads as CPU cores, and threads can cheaply and efficiently switch between work items when they are blocked.

* Runtime behavior
* Runtime contract
* Language features
## Language features
### `await` and non-blocking of threads

When written with swift concurrency primitives,

```swift
func deserializeArticles(from data: Data) throws -> [Article] { /* ... */ }
func updateDatabase(with articles: [Article], for feed: Feed) async { /* ... */ }

await withThrowingTaskGroup(of: [Article].self) { group in
    for feed in feedsToUpdate {
        group.async {
            let (data, response) = try await URLSession.shared.data(from: feed.url)
            // ...
            let articles = try deserializeArticles(from: data)
            await updateDatabase(with: articles, for: feed)
            return articles
        }
    }
}
```

Instead of handling result of networking on concurrent dispatch queue, we use a task group.  In the task group, we create child tasks for each feed that needs to be updated.

When calling async functions, we annotate with `await`.  This is an async wait.  It does not block the current thread.  Instead, function may be suspended and freed up to execute other tasks?  How does this happen?

#### non-async functions
One stack.  When threads execute a function call, a new frame is pushed onto the stack.

Can be used by the fn to store local variables, return value, etc.  One fn finishes, its stack frame is popped.  

```swift
// on Database
func save(_ newArticles: [Article], for feed: Feed) async throws -> [ID] { /* ... */ }

// on Feed
func add(_ newArticles: [Article]) async throws {
    let ids = try await database.save(newArticles, for: self)
    for (id, article) in zip(ids, newArticles) {
        articles[id] = article
    }
}

func updateDatabase(with articles: [Article], for feed: Feed) async throws {
    // skip old articles ...
    try await feed.add(articles)
}
```

Suppose that the thread calls `add` from `updateDatabase`.  At this state, the most recent stack frame is `add`.  SF stores local variables that do not need to be available across the suspension point.
Body of `add` has 1 suspension point, marked by `await`.

Local variables `id` and `article` are immediately used in the body of the for loop.  Without any suspension points in between.  So they are stored in the stack frame for `add`.

Additionally, there will be 2 async frames on the heap.  One for `updateDatabase` and one for `add`.  These store information that does need to be available across suspension points.

Note that `newArticles` is defined *before* `await`.  But needs to be available *after*.  This means that the "async frame" for `add` will keep track of `newArticles`.  

Suppose thread continues.  When `save` starts executing, the stack frame for `add` is replaced by the stack frame for `saved`.   Instead of adding new stack frames, top stackframe is replaced.  This is because any variables that are needed in the future, have been moved to the heap frames.

`save` function also gains an async frame for its use.  While the articles are being saved in the database, thread can do useful work.

Suppose `save` is suspended.  Thread is re-used to do other useful work instead of being blocked.

Since all information that is maintained across suspension points is stored on the heap, it can be used to continue execution at a later phase.  This list of async frames is the runtime representation of a continuation.

Suppose db process is complete, and some thread freed up.  Same thread, or different thread.

Suppose save function resumes on this thread.  Once it's finished executing, it returns some IDs.  Then the stack frame for `save` will again be replaced by a stackframe for `add`.  After that, thread can start executing `zip`.

zipping arrays is a non-async operation, so Swift creates a new stackframe.  async and non-async swift code can efficiently call into C and objC.  And C/objc can efficiently call **non-async** swift code.

Once zip finishes, its stackframe will be popped and execution will continue.

* `await` and non-blocking of threads
* Tracking of dependencies in Swift task model

### tracking of dependencies in Swift task model
fn can be broken up into continuations at an `await`, a.k.a. suspension point.

 
 and the part async after is a *continuation*.
Dependency tracked by the swift concurrency runtime.

Parent task may create multiple child tasks, and each child task needs to complete before a parent task can proceed.  Scope of the task group, and explicitly known to the swift compiler.

In swift, tasks can only await other tasks that are *known* to the swift runtime.  Via continuations or child tasks.  Therefore code when structured with swift concurrency primitives, give the runtime a clear understanding of dependency chain between tasks.

Therefore, we have a runtime contract that threads are **always able to make forward progress**.  We have taken advantage of the runtime contract to integrate OS support for concurrency.

## Cooperative thread pool
* Default executor for Swift
* Width limited to the number of CPU cores
	* not overcommit
* Controlled granuarity of concurrency
	* Worker threads do not block
	* Avoid thread explosion and excessive context switches

GCD
* queue per subsystem

[[Modernizing Grand Central Dispatch Usage]]
[[Concurrent Programming with GCD in Swift 3 - 16]]

It was difficult for you to get concurrency > 1 within a subsystem, without running the risk of thread explosion

IN swift
* default runtime helps you maintain the right concurrency limits

## Adoption of swift concurrency

### performance
* concurrency comes with costs
* Only write new code with swift concurrency when the cost of introducing new concurrency outweighs the cost of maanaging it.

Simply to read a bool from defaults.  The useful work done by the child task is diminished by the cost of creating and managing the task.

**We recommend profiling your code**

### await and atomicity

**No guarantee that the thread which executed the code before the `await` will execute the continuation as well**.

Breaks atomicity by voluntarily descheduling the task.

* Cannot hold locks across `await`
* Thread specific data is not preserved across `await`

### Runtime contract
Language allows to uphold a runtime contract that threads will always be able to make forward progress.

Based on this contract, we built a cooperative thread pool.

As you adopt concurrency, it's important to ensure you maintain this contract.  so that the cooperative threadpool can function optimally.

You want to use safe primitives, that make the dependencies explicit and known.  Await, actors, task groups.  Known at compile time.  Therefore the compiler enforces.

 Primitives like `os_unfair_lock`, `NSLock`, etc. are also safe in synchronous code, but there's no compiler support.  In synchronous code, *the thread holding the lock is always able to make forward progress toward releasing the lock.*  Whiel the primitive may block another thread, it does not violate the runtime contract of forward progress.
 
 Unlike swift concurrency primitives, no compiler support for these.  Your responsibility to use the primitives correctly.
 
 On the other hand, `DispatchSemaphore`, `pthread_cond`, `NScondition`, `pthread_rw_lock` etc, are unsafe to use with swift concurrency.  They hide dependency information from the Swift runtime.  Since the runtime is unaware of the dependency, it cannot make the right scheduling decisions and resolve them.
 
 In particular, don't use unsafe primitives to wait across task boundaries.  e.g. sempahore or unsafe primitive.
 
 Such a code pattern means a thread can block *indefinitely* against a sempahore, until *another* thread unblocks.  This violates the runtime contract of forward progress.
 
 To help you, we recommend testing with
 `LIBDISPATCH_COOPERATIVE_POOL_STRICT=1`.  This runs your app under a modified runtime which enforces the envariant of forward progress.  This env var can be set in xcode on the run arguments pane.
 
 > When running with this var, if you see a thread from the cooperative threadpool which appears to be hung, it indicates the use of an unsafe locking promitive

# Synchronization
Actors provide a powerful new sync primitive.

* mutual exclusion => actor state is not accessed concurrently.
* reentrancy and prioritization
* main actor

## mutual exclusion

Consider
`databaseQueue.sync { updateDatabase(articles, for: feed)}`

If the queue is not already running, there is no contention.  Here, thread is re-used.

But if the queue is running, calling thread is blocked.  This blocking behavior is what triggers threaad explosion.

Therefore, we have advised you to use async.  This is non-blocking, so even under contention, will not lead to thread explosion.  The downside is that we have to request a new thread to do the async work.  So frequent use of async can lead to excess thread wakeup and context switch.

This brings us to actors.  Actors provide cooperative threading, meaning we can reuse the thread.  In the case where it is running, we can suspend and pick up other work.

Let's focus on db and networking subsystems.  When updating the application with concurrency, serial queue is replaced with DB actor.  Concurrent queue for networking can be replaced with an actor per feed.  

These actors will run on the cooperative thread pool.  This allows actor hopping.

Run on thread from cooperative pool.  Save some articles into the db.  Suppose db is unused (uncontended case).

Thread can hop from sports feed actor, to db actor.  First, thread did not block while hopping actors.  Second, hopping did not require a different thread.  Runtime can directly suspend the work item.

Here db actor runs for awhile, but has not completed.  At this moment, suppose weather feed actor tries to save articles in the db.  

Actors ensure safety by guaranteeing mutual exclusion.  Since there is already one actor work item, the new work item is pending.  

Actors are nonblocking.  Weather feed actor will be suspended.  Executing thread is now freed up to do other work.  After awhile, db is completed.  

## Reentrancy and prioritization

How does system decide which task to resume?


Ideally, high priority work takes precedence over background work.  Actors are designed to allow the system to prioritize work well, due to reentrancy.  but to understand why this is important, first look at how GCD handles priorities.

Disaptch queues execute in strict FIFO order.  However this creates priority inversion, if high priority is later.

Serial queues work around inversion, by boosting priority of all work in the queue.  In practice, this means work in the queue will be done sooner.  However, this does not resolve the main issue which is that low priority (in terms of how they were submitted, not taking into account boosts) items will complete.

Consider the db actor.  Suppose that it is suspended, awaiting some work.  Sports feed actor.  That calls db actor.  Since db actor is uncontended, thread hops back.  Even though it has a pending work item.

This can make progress, even though one or more work items are suspended.  

Actors will finish executing.  Note that this finished before, even though was created after.  Actors can execute items in an order that is not FIFO.

1.  `await database.fetchLatestForDisplay()`
2.  Item inserted into front of queue
3.  Lower priority work follows later
4.  Directly addresses the problem of priority inversion, allowing for more effective scheduling

## main actor
Since the main thread is disjoint from the cooperative pool...

```swift
// on database actor
func loadArticle(with id: ID) async throws -> Article { /* ... */ }

@MainActor func updateUI(for article: Article) async { /* ... */ }

@MainActor func updateArticles(for ids: [ID]) async throws {
    for id in ids {
        let article = try await database.loadArticle(with: id)
        await updateUI(for: article)
    }
}
```

Each iteration of the loop, involves two context switches.  How does CPU usage look?

Since each loop iteration involves 2 context switches, we interleave.  Since substantial work is done in each iteration, it's probably fine.

However, if execution hops on and off the main actor, the overhead can add up.

If your app spends a lot of time context switching, restructure your code so that work on the main actor *is batched up*.

You can batch work by pushing the work into `loadArticles` and `updateUI` to work with arrays instead of single values.  This reduces the number of context switches.

# Wrap up

We learned how to make the system as efficient as it can be.   Cooperative thread pool, nonblocking suspensions, actor implementation.  At each step, we use some aspects of the runtime contract to improve performance of your application.

