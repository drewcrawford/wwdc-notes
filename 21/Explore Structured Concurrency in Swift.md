#swift #async 
Swift 5.5

# Structured programming

In the early days we had goto.  Today, we have structured programming.  e.g. if/then.

In Swift, we also have static scoping.  Names are only visible if they're defined in an enclosing block.  Sot he lifetime of any variables defined in a block will end in a block.

This makes control flow and variable lifetime easy to understand.  More generally, structured control flow can be sequenced and nested together naturally.  This lets you read your entire program top to bottom.

Those are the fundamentals of structured programming.  Easy to take for granted.



```swift
func fetchThumbnails(
    for ids: [String],
    completion handler: @escaping ([String: UIImage]?, Error?) -> Void
) {
    guard let id = ids.first else { return handler([:], nil) }
    let request = thumbnailURLRequest(for: id)
    let dataTask = URLSession.shared.dataTask(with: request) { data, response, error in
        guard let response = response,
              let data = data
        else {
            return handler(nil, error)
        }
        // ... check response ...
        UIImage(data: data)?.prepareThumbnail(of: thumbSize) { image in
            guard let image = image else {
                return handler(nil, ThumbnailFailedError())
            }
            fetchThumbnails(for: Array(ids.dropFirst())) { thumbnails, error in
                // ... add image to thumbnails ...
            }
        }
    }
    dataTask.resume()
}
```

You'll notice this function does not return a value when called.  The function returns result to a completion handler.  Allows caller to receive answer at a later time.

* As a consequence, it cannot use structured control flow for error handling.
* cannot use a loop.

Now let's rewrite with async/await syntax.

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        let request = thumbnailURLRequest(for: id)
        let (data, response) = try await URLSession.shared.data(for: request)
        try validateResponse(response)
        guard let image = await UIImage(data: data)?.byPreparingThumbnail(ofSize: thumbSize) else {
            throw ThumbnailFailedError()
        }
        thumbnails[id] = image
    }
    return thumbnails
}
```

Annotated with `async` and `throws`
returns a value instead of nothing
No nesting is required

[[Meet asyncawait in Swift]]

What if you're producing thumbnails for 1000s of images?  Processing each one 1 at a time is not ideal.

# Tasks in Swift
A task provides a new async context for executing code concurrently.

Each task runs concurrently
automatically scheduled to run in parallel "when it is safe and efficient to do so"
Swift checks your usage of tasks to help prevent concurrency bugs
When calling an async function **a task is not created**.  Tasks are created *explicitly*.

Multiple tpyes of tasks

## Async-let tasks

How does ordinary let work?  rvalue, lvalue.  Statements before or after.

1.  Preceding statements
2.  rvalue
3.  lvalue
4.  following statements

Only 1 flow of execution.

Since the download could take awhile, you want to start and continue doing other work until it's needed.

`async let`.  Concurrent binding.

The evaluation of a concurrent binding is quite different.

1.  Preceding statements
2.  Create child task.  Subtask of the one that created it.  Because every task represents an execution context for your program, 2 arrows will come out.
	1.  child task, this evaluates the rvalue
	2.  parent task, this binds to lvalue.  Parent task is the same that was executing the preceding statements.
3.  While 2 is taking place, we execute the following statements.
4.  Upon reaching a statement that requires the value, parent `await` on the child task.  This fulfills the placeholder for the lvalue.  Awaiting the result can give us an error.

```swift
func fetchOneThumbnail(withID id: String) async throws -> UIImage {
    let imageReq = imageRequest(for: id), metadataReq = metadataRequest(for: id)
    async let (data, _) = URLSession.shared.data(for: imageReq)
    async let (metadata, _) = URLSession.shared.data(for: metadataReq)
    guard let size = parseSize(from: try await metadata),
          let image = try await UIImage(data: data)?.byPreparingThumbnail(ofSize: size)
    else {
        throw ThumbnailFailedError()
    }
    return image
}
```
Now downloads happen in child tasks, so don't write `try await` on RHS.  This gets lifted into the parent task (lvalue).

Notice that using concurrently bound variable does not require a method call.  They have the same type as sequential binding.

These are actually part of a hierarchy called a task tree.  **Not** an implementation detail.  Influences the attributes of your tasks, like cancellation, priority, task local variables.

Whenever you make a call from one async function to another, the same task is used to execute the call. So you inherit the attributes of that call.

When you do this, tasks become a child of the parent task.  **Not a child of the specific function**.  Their lifetime *may* be scoped to the function.

A *link* between tasks enforces a rule that a parent task can only finish its work if all of its child tasks have finished.  This rule holds even in the case of abnormal control flow, which would prevent the child task from being awaited.  

e.x., we await `metadata` before `await UIImage(...)`.  If `metadata` finishes by *throwing an error*, parent task must immediately exit by throwing that error.  But, what will happen to the task performing the second download?

During abnormal exit, swift will automatically mark the unawaited task as *cancelled* and then *await* for it to finish.  **Before exiting the function**.  Marking a task as cancelled *does not stop the task*.  It simply informs the task that its results are no longer needed.

In fact, when a task is cancelled, all subtasks are automatically cancelled too.  So, if the implementation of `URLSession` creates its own tasks, those tasks will be marked for cancellation.

This guarantee is fundamental to structured concurrency.  It prevents you from leaking tasks by managing their lifetimes, much like how ARC manages the lifetime of memory.

*When* does the task finally stop?

### Cancellation is cooperative
Tasks are **not** stopped immediately when cancelled
Your code *must* check for cancellation *explicitly*
Cancellation can be checked from anywhere, whether it is async or not.
Design your code with cancellation in mind, especially if they involve long-running computation.
Users will expect computation to stop.

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        try Task.checkCancellation()
        thumbnails[id] = try await fetchOneThumbnail(withID: id)
    }
    return thumbnails
}
```

Can also `if Task.isCancelled`.

Be careful about returning partial results.  Only if the API clearly states that a partial result may be returned.  Otherwise, task cancellations could trigger a failed error, because their code requires a cmplete result even during cancellation.

## Group tasks

Async let works well when there's a fixed amount of concurrency available.

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        thumbnails[id] = try await fetchOneThumbnail(withID: id)
    }
    return thumbnails
}

func fetchOneThumbnail(withID id: String) async throws -> UIImage {
    // ...

    async let (data, _) = URLSession.shared.data(for: imageReq)
    async let (metadata, _) = URLSession.shared.data(for: metadataReq)

    // ...
}
```

Each thumbnail requires exactly 2 tasks.  2 child tasks must complete before the next iteration begins.

But what if we want this loop to fetch all thumbnails?  Then the amount of concurrency is not known statically.  For this situation, we have "task group".

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    try await withThrowingTaskGroup(of: Void.self) { group in
        for id in ids {
            group.async {
                // Error: Mutation of captured var 'thumbnails' in concurrently executing code
                thumbnails[id] = try await fetchOneThumbnail(withID: id)
            }
        }
    }
    return thumbnails
}
```

`withThrowingTaskGroup` function.  Tasks added to a group cannot outlive the scope of the block in which the group is defined.

Create child tasks in the group by calling `group.async`.  Child tasks begin immediately and execute in any order.  All tasks are implicitly awaited.

Note that as written, this has a datarace due to mutating `thumbnails`.  If two child tasks try to insert thumbnails simultaneously, it's an issue.  But Swift has static checking.

Whenever you create a new task, task creation takes an `@Sendable` closure.  Cannot capture mutable variables in the lexical context, becuase those variables could be modified after the task is launched.
Should only capture value types, actors, or classes that implement their own synchronization.

[[Protect mutable state with Swift actors]]

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    try await withThrowingTaskGroup(of: (String, UIImage).self) { group in
        for id in ids {
            group.async {
                return (id, try await fetchOneThumbnail(withID: id))
            }
        }
        // Obtain results from the child tasks, sequentially, in order of completion.
        for try await (id, thumbnail) in group {
            thumbnails[id] = thumbnail
        }
    }
    return thumbnails
}
```

This design gives the parent task the responsibility of processing the result.

Note the use of `for try await` loop, "for-await" loop.  This runs sequentially, so we can add use mutable state here.

If your own type conforms to `AsyncSequence` then you can use for-await.

[[Meet AsyncSequence]]


 ### Title
 There's a small difference in group tasks vs async let.

Suppose when iterating results, I encounter a child task that completed with error.  Because error is thrown out of the block, all tasks in the group would be implicitly cancelled and awaited.  This works just like async-let.

However, if the group goes out of scope through a normal exit.  Then, **cancellation is not implicit.**  

This behavior makes it easier to "express fork-join through a task group", because "the job will only be awaited, not cancelled".  You can also manually cancel all tasks before existing the block using `cancelAll`.

Keep in mind that no matter how you cancel a task, cancellation propagates down the tree.  async-let and group tasks are the two tasks that provide scoped, structured tasks.

## Unstructured tasks

You don't always have a hierarchy when you're adding tasks to your program.  

### Not all tasks fit a structured pattern
Some tasks need to launch from non-async contexts
Some tasks live beyodn the confines of a single scope.  e.g., start a task in response to a method call, and cancel via a different method call.

This comes up a lot when dealing with UIKit/AppKit.

[[Protect mutable state with Swift actors]]

 ```swift
 @MainActor
class MyDelegate: UICollectionViewDelegate {
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        async {
            let thumbnails = await fetchThumbnails(for: ids)
            display(thumbnails, in: cell)
        }
    }
}
```

Idea here is we bind the child to the main thread, but don't block the thread?

* Inherit actor isolation and priority of the origin context
* Lifetime is not confined to any scope
* Can be launched anwhere, even non-async functions
* Must be manually cancelled or awaited

Manually cancel/await:
```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
    var thumbnailTasks: [IndexPath: Task.Handle<Void, Never>] = [:]
    
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        thumbnailTasks[item] = async {
            defer { thumbnailTasks[item] = nil }
            let thumbnails = await fetchThumbnails(for: ids)
            display(thumbnails, in: cell)
        }
    }
    
    func collectionView(_ view: UICollectionView, didEndDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        thumbnailTasks[item]?.cancel()
    }
}
```

Note that we can access the mutable data without getting the compiler annoyed.  Our delegate is bound to the main actor and the new task inherits that, so they won't run together in parallel.  We can access stored properties without worrying about data races.

Meanwhile, if our delegate is told that the tableview is removed, we can call `.cancel()`.

Sometimes, you don't want t inherit *anything* from a context.

### Detached tasks
* Unscoped lifetime, manually cancelled and awaited
* Do not inherit anything from their originating context
* Optional parameters control priority and other traits

```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
    var thumbnailTasks: [IndexPath: Task.Handle<Void, Never>] = [:]
    
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        thumbnailTasks[item] = async {
            defer { thumbnailTasks[item] = nil }
            let thumbnails = await fetchThumbnails(for: ids)
            asyncDetached(priority: .background) {
                writeToLocalCache(thumbnails)
            }
            display(thumbnails, in: cell)
        }
    }
}
```

Instead of detaching an independent task for every background job, we can set up a task group and spawn each job as a child task of the parent.

```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
    var thumbnailTasks: [IndexPath: Task.Handle<Void, Never>] = [:]
    
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        thumbnailTasks[item] = async {
            defer { thumbnailTasks[item] = nil }
            let thumbnails = await fetchThumbnails(for: ids)
            asyncDetached(priority: .background) {
                withTaskGroup(of: Void.self) { g in
                    g.async { writeToLocalCache(thumbnails) }
                    g.async { log(thumbnails) }
                    g.async { ... }
                }
            }
            display(thumbnails, in: cell)
        }
    }
}
```

Benefits
* If we do need to cancel the task, using a task group means we can cancel all children with 1 operation.  Then this propagates automatically.
* Child tasks automatically inherit priority of their parent
* We only need to background the deatched task, so we don't need to worry about getting priority wrong

# Flavors of tasks

 ```markdown
|                    | Launched by     | Launchable from   | Lifetime             | Cancellation      | Inherits from origin               |
|--------------------|-----------------|-------------------|----------------------|-------------------|------------------------------------|
| async-let tasks    | `async let x`   | `async` functions | scoped to statement  | automatic         | priority, task-local values        |
| Group tasks        | `group.async`   | `withTaskGroup`   | scoped to task group | automatic         | priority, task-local values        |
| Unstructured tasks | `async`         | anywhere          | unscoped             | via `Task.Handle` | priority, task-local values, actor |
| Detached tasks     | `asyncDetached` | anywhere          | unscoped             | via `Task.Handle` | nothing                            |
```

# Concurrency in Swift
[[Meet asyncawait in Swift]]
[[Protect mutable state with Swift actors]]
[[Meet AsyncSequence]]
[[Swift concurrency Behind the scenes]]
