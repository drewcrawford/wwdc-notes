

Get started with noncopyable types in Swift. Discover what copying means in Swift, when you might want to use a noncopyable type, and how value ownership lets you state your intentions clearly.

Guaranteeing that values are unique are a powerful concept in programming.

# Copying
### Player as a struct - 0:52
```swift
struct Player {
  var icon: String
}

func test() {
  let player1 = Player(icon: "🐸")
  var player2 = player1
  player2.icon = "🚚"
  assert(player1.icon == "🐸")
}
```


When you copy a variable, you're copying its contents.  Changing the icon of p2 is independent of p1.
### Player as a class - 1:55
```swift
class PlayerClass {
  var icon: String
  init(_ icon: String) { self.icon = icon }
}

func test() {
  let player1 = PlayerClass("🐸")
  let player2 = player1
  player2.icon = "🚚"
  assert(player1.icon == "🐸")
}
```

Since both players refer to the same object, changing the icon changes it for both.

we can do a deep copy via a custom initializer.

### Deeply copying a PlayerClass - 3:00
```swift
class PlayerClass {
  var data: Icon
  init(_ icon: String) { self.data = Icon(icon) }

  init(from other: PlayerClass) {
    self.data = Icon(from: other.data)
  } 
}

func test() {
  let player1 = PlayerClass("🐸")
  var player2 = player1
  player2 = PlayerClass(from: player2)
  player2.data.icon = "🚚"
  assert(player1.data.icon == "🐸")
}

struct Icon {
  var icon: String
  init(_ icon: String) { self.icon = icon }
  init(from other: Icon) { self.icon = other.icon }
}
```

When designing a new type, you already have control over whether someone copies its value.  But you don't have control over whether swift makes automatic copies of it.

Copyable is a new protocol that describes the ability of a type to be automatically copied.  Like Sendable, it has no member requirements.

Everything is copyable by default.  Every type, every parameter, every protocol and associated type automatically requires Copyable.  And every boxed protocol type is automatically composed with Copyable.  Swift assumes you want the ability to copy.

There are situations where copy makes your code error-prone.


### Copyable BankTransfer - 5:10
```swift
class BankTransfer {
  var complete = false

  func run() {
    assert(!complete)
    // .. do it ..
    complete = true
  }

  deinit {
    if (!complete) { cancel() }
  }

  func cancel() { /* ... */ }
}

func schedule(_ transfer: BankTransfer, _ delay: Duration) async throws {
  if delay < .seconds(1) {
    transfer.run()
  }

  try await Task.sleep(for: delay)
  transfer.run()
}

func startPayment() async {
  let payment = BankTransfer()
  log.append(payment)
  try? await schedule(payment, .seconds(3))
}

let log = Log()

final class Log: Sendable {
  func append(_ transfer: BankTransfer) { /* ... */ }
}
```

we want to avoid copying twice.

Add a variable to track the state of the transfer?  But we won't uncover the bug until runtime.

try to solve in deinit, but here all copies aren't destroyed because retained by log, etc.

# Noncopyable types

In some situations it's better if the type were noncopyable.



### Copying FloppyDisk - 7:46
```swift
struct FloppyDisk: ~Copyable {}

func copyFloppy() {
  let system = FloppyDisk()
  let backup = consume system
  load(system)
  // ...
}

func load(_ disk: borrowing FloppyDisk) {}
```

This suppresses the default conformance to copyable.  

consume - leaves source unitialized.  Before consume, only system disk is initialized.  consume moves to backup.


### Missing ownership for FloppyDisk - 8:18
```swift
struct FloppyDisk: ~Copyable { }

func newDisk() -> FloppyDisk {
  let result = FloppyDisk()
  format(result)
  return result
}

func format(_ disk: FloppyDisk) {
  // ...
}
```

what happens here?  It's hard to tell, because function signature does not declare what ownership it needs.  With copyable you didn't have to think about this, format effectively gets a copy.  But noncopyable parameters, must declare what ownership

* consuming - takes the argument away from the caller, belongs to you
* borrowing - temporary access, read access like a let binding, under the hood how it usually works.  Can't consume or re-take a bound argument.
* inout - temporary write access to a variable in the caller.  You're allowed to consume the parameter, but you have to initialize at some point before the function ends.



### Consuming ownership - 9:00
```swift
struct FloppyDisk: ~Copyable { }

func newDisk() -> FloppyDisk {
  let result = FloppyDisk()
  format(result)
  return result
}

func format(_ disk: consuming FloppyDisk) {
  // ...
}
```

### Borrowing ownership - 9:26
```swift
struct FloppyDisk: ~Copyable { }

func newDisk() -> FloppyDisk {
  let result = FloppyDisk()
  format(result)
  return result
}

func format(_ disk: borrowing FloppyDisk) {
  var tempDisk = disk
  // ...
}
```

### Inout ownership - 9:55
```swift
struct FloppyDisk: ~Copyable { }

func newDisk() -> FloppyDisk {
  var result = FloppyDisk()
  format(&result)
  return result
}

func format(_ disk: inout FloppyDisk) {
  var tempDisk = disk
  // ...
  disk = tempDisk
}
```

Since end of my run method will automatically destroy self, i'll write `discard self`, which destroys **without calling deinit**.
### Noncopyable BankTransfer - 10:28
```swift
struct BankTransfer: ~Copyable {
  consuming func run() {
    // .. do it ..
    discard self
  }

  deinit {
    cancel()
  }

  consuming func cancel() {
    // .. do the cancellation ..
    discard self
  }
}
```



### Schedule function for noncopyable BankTransfer - 11:10
```swift
func schedule(_ transfer: consuming BankTransfer, _ delay: Duration) async throws {
  if delay < .seconds(1) {
    transfer.run()
    return
  }

  try await Task.sleep(for: delay)
  transfer.run()
}
```


# Generics

Now you can, noncopyable generics!

The universe.

Every type has happily coexisted here, whether a string or a newtype.
Protocol defines a subspace in this universe, containing types that conforms to it.

A core idea behind generics is that conformance constraints describe generic types.

Defualt constraint on generic types, that requires `T: Copyable`.  Until recently, the whole universe of types in Swift has just been Copyable in disguise. So all the types in this space can be passed into execute function.


### Overview of conformance constraints - 12:12
```swift
struct Command { }

protocol Runnable {
  consuming func run()
}

extension Command: Runnable {
  func run() { /* ... */ }
}

func execute1<T>(_ t: T) {}

func execute2<T>(_ t: T) where T: Runnable {
  t.run()
}

func test(_ cmd: Command, _ str: String) {
  execute1(cmd)
  execute1(str)

  execute2(cmd)
  execute2(str) // expected error: 'execute2' requires that 'String' conform to 'Runnable'
}
```

Now there are types without a conformance to Copyable.  ex, `BankTransfer`.

`~Copyable` is really a superset of `Copyable`.  It might be copyable, but it might not.  Tilde broadens constraints to be less specific.

`Any` has always been copyable.  When you think of `any Type`, it's copyable.



# Extensions


### Noncopyable generics: 'execute' function - 15:50
```swift
protocol Runnable: ~Copyable {
  consuming func run()
}

struct Command: Runnable {
  func run() { /* ... */ }
}

struct BankTransfer: ~Copyable, Runnable {
  consuming func run() { /* ... */ }
}

func execute2<T>(_ t: T) where T: Runnable {
  t.run()
}

func execute3<T>(_ t: consuming T) where T: Runnable, T: ~Copyable {
  t.run()
}

func test() {
  execute2(Command())
  execute2(BankTransfer()) // expected error: 'execute2' requires that 'BankTransfer' conform to 'Copyable'

  execute3(Command())
  execute3(BankTransfer())
}
```

Two ways to store noncopyable values inside another.

1.  Inside a class - we only copy a reference
2. Suppress `Copyable` on the containing type itself.
### Conditionally Copyable - 18:05
```swift
struct Job<Action: Runnable & ~Copyable>: ~Copyable {
  var action: Action?
}

func runEndlessly(_ job: consuming Job<Command>) {
  while true {
    let current = copy job
    current.action?.run()
  }
}

extension Job: Copyable where Action: Copyable {}

protocol Runnable: ~Copyable {
  consuming func run()
}

struct Command: Runnable {
  func run() { /* ... */ }
}
```

the extension tells us that Job is copyable when its action is Copyable.  We don't know if a job is copyable until we ahve a concrete type for the action.

Note that when we write extensions... they're also implicitly contrained to Copyable, including `Self` in a protocol.



### Extensions of types with noncopyable generic parameters - 19:27
```swift
extension Job {
  func getAction() -> Action? {
    return action
  }
}

func inspectCmd(_ cmdJob: Job<Command>) {
  let _ = cmdJob.getAction()
  let _ = cmdJob.getAction()
}

func inspectXfer(_ transferJob: borrowing Job<BankTransfer>) {
  let _ = transferJob.getAction() // expected error: method 'getAction' requires that 'BankTransfer' conform to 'Copyable'
}


struct Job<Action: Runnable & ~Copyable>: ~Copyable {
  var action: Action?
}

extension Job: Copyable where Action: Copyable {}

protocol Runnable: ~Copyable {
  consuming func run()
}

struct Command: Runnable {
  func run() { /* ... */ }
}

struct BankTransfer: ~Copyable, Runnable {
  consuming func run() { /* ... */ }
}
```

### Cancellable for Jobs with Copyable actions - 20:14
```swift
protocol Cancellable {
  mutating func cancel()
}

extension Job: Cancellable {
  mutating func cancel() {
    action = nil
  }
}
```

### Cancellable for all Jobs - 21:00
```swift
protocol Cancellable: ~Copyable {
  mutating func cancel()
}

extension Job: Cancellable where Action: ~Copyable {
  mutating func cancel() {
    action = nil
  }
}
```


# Wrap up
* noncopyable types improve ownership, etc.
* Optional, UnsafePointer, and Result
* Swift Evolution proposals

[[Modern Swift API design - 19]]

# Resources
* [Copyable](https://developer.apple.com/documentation/Swift/Copyable)
* [Forum: Programming Languages](https://developer.apple.com/forums/topics/programming-languages-topic?cid=vf-a-0010)
* [Swift Evolution: Borrowing and consuming pattern matching for noncopyable types](https://github.com/apple/swift-evolution/blob/main/proposals/0432-noncopyable-switch.md)
* [Swift Evolution: Noncopyable Generics](https://github.com/apple/swift-evolution/blob/main/proposals/0427-noncopyable-generics.md)
* [Swift Evolution: Noncopyable Standard Library Primitives](https://github.com/apple/swift-evolution/blob/main/proposals/0437-noncopyable-stdlib-primitives.md)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10170/4/993789F1-AF44-4E20-8C66-BF59DAC6C1F6/downloads/wwdc2024-10170_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10170/4/993789F1-AF44-4E20-8C66-BF59DAC6C1F6/downloads/wwdc2024-10170_sd.mp4?dl=1)