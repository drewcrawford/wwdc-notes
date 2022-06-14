#async #swift 

# Functions: synchronous and asynchronous
Your thread is blocked in synchronous functions.
Your thread can't do anything else until it finishes.

If you call async function, your thread is free to do other work.  

SDK provides many async functions.  Some use completion handlers, delegate callbacks, and many are marked `async`.

When you call them, it unblocks your thread quickly.  That allows the thread to do other things while long-running work completes.

```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(nil, error)
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(nil, FetchError.badID)
        } else {
            guard let image = UIImage(data: data!) else {
                completion(nil, FetchError.badImage)
                return
            }
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumbnail in
                guard let thumbnail = thumbnail else {
                    completion(nil, FetchError.badImage)
                    return
                }
                completion(thumbnail, nil)
            }
        }
    }
    task.resume()
}
```

Easy to forget to invoke completion handler, leaving caller in the lurch.  Caller will never be notified.

We can do better with async/await.

```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)  
    let (data, response) = try await URLSession.shared.data(for: request)
    guard (response as? HTTPURLResponse)?.statusCode == 200 else { throw FetchError.badID }
    let maybeImage = UIImage(data: data)
    guard let thumbnail = await maybeImage?.thumbnail else { throw FetchError.badImage }
    return thumbnail
}
```

Thread is unblocked to do other work more often.

Errors handled with try/catch.

Like `try` is needed to call `throws`, `await` is needed to call `async`.

You'll need to put `try` before `await`.

## How it's implemented
Not just functions can be async.  Properties can be too.  Also initializers.

*Only read-only properties can be async.*

Explicit getter is required to mark the property async.  

If the property is both `async` and `throws`, `async` goes before `throws`.

# Async sequences
```swift
for await id in staticImageIDsURL.lines {
    let thumbnail = await fetchThumbnail(for: id)
    collage.add(thumbnail)
}
let result = await collage.draw()
```

[[Meet AsyncSequence]]

[[Explore structured concurrency in Swift]]

# suspend
What does it mean for an `async` function to suspend?

Normal call: You hand control of the thread over to the function.  Thread will be fully occupied doing work on behalf of that one function until it finishes.  Might be in the body of the fucntion, or in other function it calls.  Eventually, that functioon will finish either by returning a value or throwing.  When it does, it hands control back to your function.  That's the only way a normal function can give up control of a thread.  Only return to caller.

Async function: when it's done, will finish and return to your function.  But unlike a normal function, can suspend.

When you call async you give control of the thread to it.  Once it's running, the function can suspend.  This gives up control of the thread.  But instead of giving control back to caller, it instead gives it to the system.  Your function is suspended too.

Suspending is the function's way of telling the system "you decide what's most important".  Once the function suspends, the system is free to use the thread for other work.  Eventually, the system resumes the async function.  It can suspend itself as many times as it needs to.

On the other hand, it may not need to suspend itself at all.  While async *may* suspend, it doesn't necessarily mean that it *will* suspend.  Just because you see `await` doesn't mean the function will *definitely* suspend there.  But eventually, the function finishes, handing control of the thread back to your function, along with a value or an error.

You need to be aware that the state of your app can change *dramatically* at suspension points.

This is also true at completion handlers, but since `await` is smaller, keep this in mind.

Function may resume onto a different thread.

[[Protect mutable state with Swift actors]]

# Async/await facts
* `async` enables function to suspend
	* When a function suspends, its callers suspend too.  So its callers must be async as well.
* `await` marks when an `async` function *may* suspend execution.
* Other work can happen during a suspension.  Thread is **not** blocked.
	* Your app state can change a great deal during suspension
* Once an awaited async call completes, execution resumes after the `await`.

# Adopting async/await
## Testing async code.
```swift
class MockViewModelSpec: XCTestCase {
    func testFetchThumbnails() throws {
        let expectation = XCTestExpectation(description: "mock thumbnails completion")
        self.mockViewModel.fetchThumbnail(for: mockID) { result, error in
            XCTAssertNil(error)
            expectation.fulfill()
        }
        wait(for: [expectation], timeout: 5.0)
    }
}
```

What used to be a tedious process of expectations etc., is now await.

```swift
class MockViewModelSpec: XCTestCase {
    func testFetchThumbnails() async throws {
        XCTAssertNoThrow(try await self.mockViewModel.fetchThumbnail(for: mockID))
    }
}
```

## Application code
### Calling async functions
```swift
struct ThumbnailView: View {
    @ObservedObject var viewModel: ViewModel
    var post: Post
    @State private var image: UIImage?

    var body: some View {
        Image(uiImage: self.image ?? placeholder)
            .onAppear {
                async {
                    self.image = try? await self.viewModel.fetchThumbnail(for: post.id)
                }
            }
    }
}
```

Cannot call async functions in contexts that are not async.

`onAppear` takes a plain, non-async closure.  Need to bridge the gap between sync/async worlds.

The `async` task option, packages up closure work and sends to the system for immediate execution, like the `async` function on a global dispatch queue.

Async code can be called from inside a sync context.  That satisfies the compiler.

async tasks are part of a family of APIs.

[[Explore structured concurrency in Swift]]
[[Discover concurrency in SwiftUI]]

# Async alternatives
* Swift concurrency should be adopted gradually
* Offer async alternatives to completion handler APIs
* Xcode's async refactoring actions can help

# SDK
SDK offers hundreds of APIs that take completion handlers.  As of Swift 5.5, we convert these to async.  Swift compiler automatically looks at imported objc code and provides an async alternative.

Many delegate APIs also include methods that pass a completion handler to you.  Calling the handler cooperatively informs the framework that a task is complete.

```swift
import ClockKit

extension ComplicationController: CLKComplicationDataSource {
    func currentTimelineEntry(for complication: CLKComplication) async -> CLKComplicationTimelineEntry? {
        let date = Date()
        let thumbnail = try? await self.viewModel.fetchThumbnail(for: post.id)
        guard let thumbnail = thumbnail else {
            return nil
        }

        let entry = self.createTimelineEntry(for: thumbnail, date: date)
        return entry
    }
}
```

> We recommend omitting `get` in async contexts where the value is not immediately returned.

[[Use asyncawait with URLSession]]
[[Bring Core Data concurrency to Swift and SwiftUI]]
[[What's new in AVFoundation]]
[[21/What's new in AppKit]]

# Async alternatives and continuations

There will inevitably be places in your code where you need to create async alternatives yourself.

```swift
// Existing function
func getPersistentPosts(completion: @escaping ([Post], Error?) -> Void) {       
    do {
        let req = Post.fetchRequest()
        req.sortDescriptors = [NSSortDescriptor(key: "date", ascending: true)]
        let asyncRequest = NSAsynchronousFetchRequest<Post>(fetchRequest: req) { result in
            completion(result.finalResult ?? [], nil)
        }
        try self.managedObjectContext.execute(asyncRequest)
    } catch {
        completion([], error)
    }
}

// Async alternative
func persistentPosts() async throws -> [Post] {       
    typealias PostContinuation = CheckedContinuation<[Post], Error>
    return try await withCheckedThrowingContinuation { (continuation: PostContinuation) in
        self.getPersistentPosts { posts, error in
            if let error = error { 
                continuation.resume(throwing: error) 
            } else {
                continuation.resume(returning: posts)
            }
        }
    }
}
```

We need to call further on to a callback like API from our async function, but this is a problem.  callers are in a suspended state, need to make sure to resume them at the right point in time.

## How does suspend/resume work?

1.  `getPersistentPosts` calls into coredata
2.  CoreData calls completion handler and passes result into invocation

What's missing is a bridge to await the completion handler and resume with results.  **continuation**.

Caller of method `await` result
provides a closure to specify what to do next
when call completes, "resume"

This kind of cooperative execution is exactly the way swift async works.  Swift provides a feature to create, manage, resume continuations in a high level.

```swift
func persistentPosts() async throws -> [Post] {       
    typealias PostContinuation = CheckedContinuation<[Post], Error>
	
	//This "lifts" a completion block (with error)
	//"up" to throwing function.  There's also `withCheckedContinuation` if there's no error.
	//this bridges between asyncawait.
    return try await withCheckedThrowingContinuation { (continuation: PostContinuation) in
        self.getPersistentPosts { posts, error in
            if let error = error { 
			//resume provides the missing length we need to unsuspend calls awaiting result.
                continuation.resume(throwing: error) 
            } else {
                continuation.resume(returning: posts)
            }
        }
    }
}
```

This bridges between existing and new styles.

Things to keep in mind.

## Checked continuations
* Continuations must be resumed *exactly once* on every path
* Discarding the continuation without resuming **is not allowed**
* Swift will check your work
	* Resume multiple times => error
	* Discard continuation => **warning**

### Delegate example

```swift
class ViewController: UIViewController {
    private var activeContinuation: CheckedContinuation<[Post], Error>?
    func sharedPostsFromPeer() async throws -> [Post] {
        try await withCheckedThrowingContinuation { continuation in
            self.activeContinuation = continuation
            self.peerManager.syncSharedPosts()
        }
    }
}

extension ViewController: PeerSyncDelegate {
    func peerManager(_ manager: PeerManager, received posts: [Post]) {
        self.activeContinuation?.resume(returning: posts)
        self.activeContinuation = nil // guard against multiple calls to resume
    }

    func peerManager(_ manager: PeerManager, hadError error: Error) {
        self.activeContinuation?.resume(throwing: error)
        self.activeContinuation = nil // guard against multiple calls to resume
    }
}
```

If your delegate API is called many times, or not at all.  It is critical to resume any active continuation, exactly once.

To learn more,

[[Swift concurrency Behind the scenes]]

# Summary
* Async/await code is simple, easy, and safe
* Hundreds of new async APIs
* Automatic async legacy API imports


