#async #swift 

# Data races make concurrency hard
When two threads concurrenlt access the same data
One of them is a write

```swift
class Counter {
    var value = 0

    func increment() -> Int {
        value = value + 1
        return value
    }
}

let counter = Counter()

asyncDetached {
    print(counter.increment()) // data race
}

asyncDetached {
    print(counter.increment()) // data race
}
```

* non-local reasoning to understand
* non-deterministic because OS scheduler is involved

One way to avoid data races is to avoid shared mutable state.  

`let` properties of value types are safe because they're immutable.

```swift
var array1 = [1, 2]
var array2 = array1

array1.append(3)
array2.append(4)

print(array1)        // [1, 2, 3]
print(array2)        // [1, 2, 4]
```

Majority of types in swift standard library have value semantics.  

Value semantics solve all of our data races.

```swift
struct Counter {
    var value = 0

    mutating func increment() -> Int {
        value = value + 1
        return value
    }
}

let counter = Counter()

asyncDetached {
    var counter = counter
    print(counter.increment()) // always prints 1
}

asyncDetached {
    var counter = counter
    print(counter.increment()) // always prints 1
}
```

Now it seems tempting to just change the counter variable to a var to get it mutable, but that leaves us again with a race condition since the counter is referenced with both concurrent tasks.

Compiler does not allow this.  We could instead make the counter a local variable (ex above)

But the behavior is not what we want anymore.  There are still cases where shared mutable state is required.

With shared mutable state, we need some form of synchronization to ensure we don't have data races.

Various primitives exists
* atomics
* locks
* serial dispatch queues

All require careful discipline to use correctly.

# Actors
Actors provide synchronization for shared mutable state
Actors isolate their state from the program
Only access through the actor
When you go through the actor, the sync mechanism ensures that no other code is accessing the state
This ensures mutually-exclusive access to its state.

You can't forget to perform the synchronization.

```swift
actor Counter {
    var value = 0

    func increment() -> Int {
        value = value + 1
        return value
    }
}

let counter = Counter()

asyncDetached {
    print(await counter.increment())
}

asyncDetached {
    print(await counter.increment())
}
```

* Similar capabilities to structs, enums, and classes
* Reference type
* Synchronization and data isolation set actors apart

Like classes, they are reference types because the purpose of actors are shared mutable state.

The actor will ensure the value isn't accessed concurrently.
Will run to completion without new code executing.  Eliminates the potential for data races.

How does this work?

1.  One gets their first
2.  the other has to wait

But how do we know?

Whenever you interact with an actor, you do so with `await`.  If the actor is busy, your code will suspend.  When the actor becomes free, it will wake up and resume execution.

This indicates that the async call might involve such a suspension.

## Synchronous interaction within an actor
* calls within actor are synchronous
* synchronous code always runs uninterrupted

```swift
extension Counter {
    func resetSlowly(to newValue: Int) {
        value = 0
        for _ in 0..<newValue {
            increment()
        }
        assert(value == newValue)
    }
}
```

This can directly access the actor state.  This can also synchronously call methods on the actor.  There's no `await` required because we're already on the actor.  Sync code always runs to completion without being interrupted.  We can reason sequentially.

However, actors often run with other actors.  

# Actor reentrancy

```swift
actor ImageDownloader {
    private var cache: [URL: Image] = [:]

    func image(from url: URL) async throws -> Image? {
        if let cached = cache[url] {
            return cached
        }

        let image = try await downloadImage(from: url)

        // Potential bug: `cache` may have changed.
        cache[url] = image
        return image
    }
}
```

Stores downloaded images in a cache.  

Whenever an await occurs, it means that the function can be suspended at this point.

When our function resumes, the program state will have changed.  Important to ensure that you haven't made assumptions that may not hold after.

Even though the cache was already populated with an image, we now get a different image for a URL.  Whereas we expect that we get the same image until we clear the cache.

We don't have any low-level data races, but since we carried assumptions about the state across an `await` we still have a bug.

One solution is to check the cache again after the `await`.

A better solution is to avoid the download entirely. 

```swift
actor ImageDownloader {

    private enum CacheEntry {
        case inProgress(Task.Handle<Image, Error>)
        case ready(Image)
    }

    private var cache: [URL: CacheEntry] = [:]

    func image(from url: URL) async throws -> Image? {
        if let cached = cache[url] {
            switch cached {
            case .ready(let image):
                return image
            case .inProgress(let handle):
                return try await handle.get()
            }
        }

        let handle = async {
            try await downloadImage(from: url)
        }

        cache[url] = .inProgress(handle)

        do {
            let image = try await handle.get()
            cache[url] = .ready(image)
            return image
        } catch {
            cache[url] = nil
            throw error
        }
    }
}
```

Guarantees forward-progress.  
* Perform mutation of actor state in synchronous code.  Ideally in a sync function.
* State changes can involve temporarily putting our actor into an inconsistent state
* Expect that the actor state could change during suspension
* Check your assumptions afte ran `await`.

# Actor isolation
## protocol conformances
Fundamental to the behavior of actor types.
Like other types, actors can conform to protocols so long as they can satisfy the requirements of the protocol.

```swift
actor LibraryAccount {
    let idNumber: Int
    var booksOnLoan: [Book] = []
}

extension LibraryAccount: Equatable {
    static func ==(lhs: LibraryAccount, rhs: LibraryAccount) -> Bool {
        lhs.idNumber == rhs.idNumber
    }
}
```

Because the method is `static`, there is no `self` instance so it is not isolated to the actor.  Here, there are two instances of the actor, and this method is outside of *both* of them.

That's ok!  Because the implementation is only accessing *immutable* state.

Ok, how about hashable

```swift
actor LibraryAccount {
    let idNumber: Int
    var booksOnLoan: [Book] = []
}

extension LibraryAccount: Hashable {
    nonisolated func hash(into hasher: inout Hasher) {
        hasher.combine(idNumber)
    }
}
```

This conformance isn't allowed.  Conforming to `hashable` means that this func can be called from outside the actor.   But `hash(into)` is not `async`.  So there's no way to maintain actor isolation.

To fix this, we can make this method `nonisolated`.

```swift
actor LibraryAccount {
    let idNumber: Int
    var booksOnLoan: [Book] = []
}

extension LibraryAccount: Hashable {
    nonisolated func hash(into hasher: inout Hasher) {
        hasher.combine(idNumber)
    }
}
```

Here, the method is "outside" the actor, even though it is, "syntactically" inside the actor.  This means that it can satisfy the sync requirement from the `Hashable` protocol.

Because it's outside the actor, they cannot reference mutable state on the actor.  This method is fine because it's referring to immutable id.  But if we tried to refer to mutable, we can't refer to the mutable state from the *outside*.

## closures
Like functions, closures might be actor-isolated or nonisolated.  In this example

```swift
extension LibraryAccount {
    func readSome(_ book: Book) -> Int { ... }
    
    func read() -> Int {
        booksOnLoan.reduce(0) { book in
            readSome(book)
        }
    }
}
```

Call to `reduce` involves a closure.  There is no `await` here, this closure *within the actor-isolated function `read`* is itself actor-isolated.  We know since closure is non-escaping, that it's fine.

```swift
extension LibraryAccount {
    func readSome(_ book: Book) -> Int { ... }
    func read() -> Int { ... }
    
    func readLater() {
        asyncDetached {
            await read()
        }
    }
}
```

Detached task will not be on the actor.  So this is not isolated, and must use `await`.

## data

What the book type actually is?  

```swift
actor LibraryAccount {
    let idNumber: Int
    var booksOnLoan: [Book] = []
    func selectRandomBook() -> Book? { ... }
}

struct Book {
    var title: String
    var authors: [Author]
}

func visit(_ account: LibraryAccount) async {
    guard var book = await account.selectRandomBook() else {
        return
    }
    book.title = "\(book.title)!!!" // OK: modifying a local copy
}
```

value type is a good choice.  Here we get a copy of the book, changes won't affect the actor and vice versa.

However, if we turn the book into a class, things are different.  Our library account actor now references instances of the book class.  What happens when we call the method to select a random book?

Now we have a reference into the mutable state of the actor

```swift
actor LibraryAccount {
    let idNumber: Int
    var booksOnLoan: [Book] = []
    func selectRandomBook() -> Book? { ... }
}

class Book {
    var title: String
    var authors: [Author]
}

func visit(_ account: LibraryAccount) async {
    guard var book = await account.selectRandomBook() else {
        return
    }
    book.title = "\(book.title)!!!" // Not OK: potential data race
}
``` 

...which is shared *outside* of the actor.  Now if we update the title of the book, the modification happens inside state that is accessible to the actor.

Because the `visit` method is not on the actor, **this modification could end up being a data race.**

* Value types and actors are safe to use concurrently
* But classes can pose problems

We have a name for types that are safe to use concurrently

`@Sendable`

# `Sendable` types
Sendable types are safe to share concurrently.
Many different types are `Sendable`
* Value types
* Actor types
* Immutable classes
* Internally-synchronized class

**Most classes are not `Sendable`**

There is an `@Sendable` function type as well.

## Checking `Sendable`
`Sendable` describes a common but not universal property of types
Swift will eventually prevent non-Sendable types from being shared

**At that point it will become an error to pass a non-Sendable type across boundaries**

How does one know that a type is `Sendable`?

**Add a conformance**.  Swift will then check to make sure your type makes sense.  Book struct is sendable if all of its properties are of sendable type.

```swift
struct Book: Sendable {
    var title: String
    var authors: [Author]
}
```

What if `Author` is a class?  

For generic types, whether they are sendable depends on generic arguments.  Can use conditional conformance to propagate sendable when appropriate

```swift
struct Pair<T, U> {
    var first: T
    var second: U
}

extension Pair: Sendable where T: Sendable, U: Sendable {
}
```

An array of sendable types is itself sendable.  We encourage you to introduce sendable conformances.  Use those types within your actors, and when Swift begins to enforce sendable across actors, you'll be ready

## `@Sendable` functions
`@Sendable` function types conform to the `Sendable` protocol
`@Sendable` places restrictions on closures
* No mutable captures
* Captures must be of `Sendable` type
* Cannot be both syncronous and actor-isolated.  (That would allow to run code on the actor, from the outside.)



`asyncDetached` takes `(_ operation: @Sendable)`.

Used to indicate where concurrent execution can occur.

Because the closure for the detached task is `Sendable` we know that it should not be isolated to the actor.  

Helps maintain actor-isolation by checking that mutable state isn't shared across actors and cannot be modified concurrently.

# Main actor
## Interacting with the main thread
* UI rendering
* Main run loop to process events

Need to iteract with the main thread where you have to, but get off quickly.

```swift
func checkedOut(_ booksOnLoan: [Book]) {
    booksView.checkedOutBooks = booksOnLoan
}

// Dispatching to the main queue is your responsibility.
DispatchQueue.main.async {
    checkedOut(booksOnLoan)
}
```

MT is a lot like an actor.  if you know you're already on the MT, you can safely access and update your UI state.  Otherwise, you need to interact async.

Special actor to describe the main thread.

```swift
@MainActor func checkedOut(_ booksOnLoan: [Book]) {
    booksView.checkedOutBooks = booksOnLoan
}

// Swift ensures that this code is always run on the main thread.
await checkedOut(booksOnLoan)
```

This differs from the normal actor in 2 important ways
* All sync through main dispatch queue
	* From a runtime perspective, interchangeable with `DispatchQueue.main`
* Code and data that needs to be on MT is everywhere.  With Swift concurrency, you can mark with `@MainActor`.  If you call from outside, you need to `await`.  By marking code, there's no more guesswork about `DispatchQueue.main`.

Types can be placed on the Main Actor
* Implies that all methods and properties of the type are `MainActor`

```swift
@MainActor class MyViewController: UIViewController {
    func onPress(...) { ... } // implicitly @MainActor

    nonisolated func fetchLatestAndDisplay() async { ... } 
}
```

Types can be placed on the main actor
* Implies that all methods and properties of the type are MainActor
* opt out with `nonisolated` on individual members

Can architect your app to ensure safe, correct use of concurrency.

# wrap up
Use actors to synchronize access to mutable state
Design for actor reentrancy
Prefer value types and actors to eliminate data races
Leverage the main actor to protect UI interactions
[[Swift concurrency update a sample app]]
[[Swift concurrency Behind the scenes]]



