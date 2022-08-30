What is swift #async concurrency?
* #async /await
* structured concurrency
* actors
A model for data-race-free concurrency.

The sea of concurrency is unpredictable.  But with you at the helm, and swift hepling you navigate the waters, you can produce amazing things.

# Task isolation
A task performs specific jobs from start to finish (boats)

```swift
Task.detached {
  let fish = await catchFish()
  let dinner = await cook(fish)
  await eat(dinner)
}
```

Work can be suspended at await ops.
Self-contained.  Each task has its own resources, os it can operate by itself, independently of other boats.

If your boats are completely independent, we have concurrency without data races.  But not useful without some way to communicate.

Boats meat on the open sea and share a pineapple?

```swift
enum Ripeness {
  case hard
  case perfect
  case mushy(daysPast: Int)
}

struct Pineapple {
  var weight: Double
  var ripeness: Ripeness
  
  mutating func ripen() async { … }
  mutating func slice() -> Int { … }
}
```

Swift has preferred value types for this reason.  Mutations have only local effects.  Helps value types maintain isolation.

```swift
final class Chicken {
  let name: String
  var currentHunger: HungerLevel
  
  func feed() { … }
  func play() { … }
  func produce() -> Egg { … }
}
```

Copying a reference type gives you a reference to the specific object.  Once our boats have gone their separate ways, we have a problem.  Both boats are doing their work ocncurrency, but they are not inependent because they sahre the same object.

We need a way to know it was safe to share pineapples, but not chickens.  And checking.

```swift
protocol Sendable { }
```

can cross an isolation domain.

Use conformance to specify which types are Sendable.

```swift
struct Pineapple: Sendable { … } //conforms to Sendable because its a value type
class Chicken: Sendable { } // cannot conform to Sendable because its an unsynchronized reference type.
```

Checking sendable across task boundaries.

```swift
// will get an error because Chicken is not Sendable
let petAdoption = Task {
  let chickens = await hatchNewFlock()
  return chickens.randomElement()!
}
let pet = await petAdoption.value
```

Specifies that the result type of a task must conform to Sendable.  Use sendable when you have generic parameters whose values will be passed across isolation domains.
```swift
struct Task<Success: Sendable, Failure: Error> {
  var value: Success {
    get async throws { … }
  }
}
```

checking.
```swift
enum Ripeness: Sendable {
  case hard
  case perfect
  case mushy(daysPast: Int)
}

struct Pineapple: Sendable {
  var weight: Double
  var ripeness: Ripeness
}
```

enums and structs generally define value types which copy instance data.  Can be sendable, so long as all instance data is also sendable.

Propagated through collections and ther generic types using conditional conformances.  Inferred by the compiler for nonpublic type.

```swift
//contains an array of Sendable types, therefore is Sendable
struct Crate: Sendable {
  var pineapples: [Pineapple]
}
```

collection of Chickens cannot be sendable.

```swift
//stored property 'flock' of 'Sendable'-conforming struct 'Coop' has non-sendable type '[Chicken]'
struct Coop: Sendable {
  var flock: [Chicken]
}
```

Classes are reference types, so they can only be made sendable under very rare circumstances.

```swift
//Can be Sendable if a final class has immutable storage
final class Chicken: Sendable {
  let name: String
  var currentHunger: HungerLevel //'currentHunger' is mutable, therefore Chicken cannot be Sendable
}
```

Possible to do internal synchronization.
```swift
//@unchecked can be used, but be careful!
class ConcurrentCache<Key: Hashable & Sendable, Value: Sendable>: @unchecked Sendable {
  var lock: NSLock
  var storage: [Key: Value]
}
```

Task creation starts a new, independent task.

```swift
let lily = Chicken(name: "Lily")
Task.detached {@Sendable in
	lily.feed()
}
```

capture values from the original task, and pass them into the new task.  So we need `Sendable` checking to ensure we don't have data races.  If we do send a non-sendable type, we get an error.

inferred to be `@Sendable` closure.  We can write that explicitly `@Sendable in`.

```swift
struct Task<Success: Sendable, Failure: Error> {
  static func detached(
    priority: TaskPriority? = nil,
    operation: @Sendable @escaping () async throws -> Success
  ) -> Task<Success, Failure>
}
```

Normally, function types cannot conform to protocols, but Sendable is special.  Compiler validates semantics.

Taskasa re isolated, independently exectuing async operations
sendable checking ensures tasks remain isolated
use sendable ocnstraints whenever values need to be shared.

But without any notion of shared mutable data, hard to coordinate.

# Actor isolation
Provide a way to isolate state that can be accessed by different tasks, in a coordinated manner that eliminates data races.

Islands.

```swift
actor Island {
  var flock: [Chicken]
  var food: [Pineapple]

  func advanceTime()
}
```

Self-contained with its own state that is isolated.  To access that state, your code needs to be running on the island.  

Only one task can execute on an actor at a time.

```swift
func nextRound(islands: [Island]) async {
  for island in islands {
    await island.advanceTime()
  }
}
```

Other tasks must "a"wait their turn
get it... await their turn

Non-Sendable data cannot be shared between a task and an actor.

```swift
//Both examples cannot be shared
await myIsland.addToFlock(myChicken)
myChicken = await myIsland.adoptPet()
```

Actors isolate all their internal mutable state.
All actor types are Sendable.

```swift
actor Island {
  var flock: [Chicken]
  var food: [Pineapple]

  func advanceTime() {
    let totalSlices = food.indices.reduce(0) { (total, nextIndex) in
      total + food[nextIndex].slice()
    }

    Task {
      flock.map(Chicken.produce)
    }

    Task.detached {
      let ripePineapples = await food.filter { $0.ripeness == .perfect }
      print("There are \(ripePineapples.count) ripe pineapples on the island")
    }
  }
}
```

actor isolation is determined by context.  
* instance properties are isolated
* instance methods are isolated (even in extensions)
* closures that are not Sendable, such as `.reduce`, stay on the actor, and are actor-isolated.
* task initializer also inherits actor isolation from its context.  Task will be scheduled on the same actor.
* *detached* task does *not* inherit actor isolation.
	* nonisolated code

functions within actors can be explicitly marked non-isolated

```swift
extension Island {
  nonisolated func meetTheFlock() async {
    let flockNames = await flock.map { $0.name }
    print("Meet our fabulous flock: \(flockNames)")
  }
}
```

Non-isolated async code executes on the cooperative pool.

Compiler detects the potential data race where instance of non-sendable chicken is tryign to leave the island.

Non-isolated *synchronous* code executes wherever it is called.

```swift
func greet(_ friend: Chicken) { }

extension Island {
  func greetOne() {
    if let friend = flock.randomElement() { 
      greet(friend)
    }
  }
}
```

nonisolated "async" operation that calls greet.  Greet will run there on the open sea.  

Most code is synchronous, not isolated to an actor, and operates on parameters.  Stays in domain where it is called.
```swift
func greet(_ friend: Chicken) { }

func greetAny(flock: [Chicken]) async {
  if let friend = flock.randomElement() { 
    greet(friend)
  }
}
```

* each actor instance is isolated from everything else in the program
* only one task can execute on an actor at a time
* `Sendable` checking occurs whenever you *enter* or *exit* an actor
* actors themselves are `Sendable`.

## `@MainActor`

Big island in the middle of the sea.  The main thread, where all the drawing, and interaction occurs.  You need to run the code on the main actor's island.  Important for your UI, that maybe we should call it the U*island*.  Get it?  UIsland?

* Main actor carries a lot of state related to the program's UI
* Lots of UI framework code and app code needs to run on it
* BUT it can still only run a single job at a time
	* don't overload it

```swift
@MainActor func updateView() { … }

Task { @MainActor in
	// …
  view.selectedChicken = lily
}

nonisolated func computeAndUpdate() async {
  computeNewValues()
  await updateView()
}
```

swift compiler guarantees it only runs on the main thread.  If one calls `updateView` from a different context, it will need to introduce an `await`.

also be applied to types.  In which case, the instances of the types will be isolated to the main actor.

Like normal actors, references to `@MainActor` are `: Sendable`.  This makes `@MainActor` suitable for UIView and UIViewController.  Share a reference to your VC to your other task, and they can asynchronously call back into the VC to post results.  This effects your app architecture.
```swift
@MainActor
class ChickenValley: Sendable {
  var flock: [Chicken]
  var food: [Pineapple]

  func advanceTime() {
    for chicken in flock {
      chicken.eat(from: &food)
    }
  }
}
```

* Views, VCs on the main actor
* separate actors from main actor
* tasks shuttle

[[Visualize and optimize Swift concurrency]]


# Atomicity
The goal of the swift ocncurrency model is to eliminate data races.  *Low-level* data races.  Still need to reason about atomicity at a high level.
* actors run one task at a time
* when you stop running on an actor, it can run other tasks
	* ensures program makes forwardprogress
	* consider await statements carefully

```swift
func deposit(pineapples: [Pineapple], onto island: Island) async {
   var food = await island.food
   food += pineapples
   await island.food = food
}
```

```swift
await island.food.takeAll()
```
atomicity violated across suspension point.

Instead, rewrite as synchronous code on the actor.
```swift
extension Island {
   func deposit(pineapples: [Pineapple]) {
      var food = self.food
      food += pineapples
      self.food = food
   }
}
```

Think transactionally
* identify synchronous operations that can be interleaved
	* leave actor in good state
* Keep async actor operations simple


# Ordering
many things are happening at once.  The other can vary from one execution to the next.
Programs often rely on handling events in a consistent order.
* user input
* mesasges from a server
* Effects of each event should appear in the order that they happened

*actors* are nto strictly FIFO
Actors execute the **highest priority work first**.
Important semantic difference vs serial Dispatch queues.

Tools for ordering
* tasks run code in order
* `AsyncStreams` deliver elements in order

```swift
for await event in eventStream {
  await process(event)
}
```

Isolation with `Sendable` checking eliminates data races
Staged rollout of `Sendable` checking throughout Swift 5 for completion in Swift 6.

Default - minimal.  Only checks explicit `Sendable` conformances.

```swift
import FarmAnimals
struct Coop: Sendable {
  var flock: [Chicken]
}
```

To move further, enable Targeted.  This enables sendable checking for code that has adopted concurrency features.

Sometimes, non-sendable types occur in another module.  Or in your own module that you haven't gotten around to.  `@preconcurrency` disables warnings for types from that module.  This silences sendable warnings within this source file.  AT some point, that module will get updated with conformances.
1.  Either Chicken becomes sendable somehow, in which case the `@preconcurrency`
2. Chicken will be known to be non-sendable, in which case the warning will return.

```swift
@preconcurrency import FarmAnimals

func visit(coop: Coop) async {
  guard let favorite = coop.flock.randomElement() else {
    return
  }

  Task {
    favorite.play()
  }
}
```

If you'd like to see everywhere that races could occur, there's one more option. Complete – approximates Swift 6 data-race elimination.  Does so for all code in the module.

Here, we're not using concurrency features.  But async operation is known to take Sendable closure, so.

```swift
import FarmAnimals

func doWork(_ body: @Sendable @escaping () -> Void) {
  DispatchQueue.global().async {
    body()
  }
}

func visit(friend: Chicken) {
  doWork {
    friend.play()
  }
}
```

Now we can see that `visit` is the source of a data race.

# the road to data-race safety
* Enable stricter concurrency checking one module at a time
* Use `@preconcurrency` to suppress warnings from another module.




https://developer.apple.com/documentation/Swift/concurrency
https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html