#swift 
Join us for an update on Swift. We'll show you how APIs are becoming more extensible and expressive with features like parameter packs and macros. We'll also take you through improvements to interoperability and share how we're expanding Swift's performance and safety benefits everywhere from Foundation to large-scale distributed programs on the server.

# Swift project update
swift.org/swift-evolution

language group.

Sometimes, individual language proposals come apart as part of a wider theme.  
Ecosystem steering group.  
# Expressive code

if/else/switch can be used as expressions.
##  Hard-to-read compound ternary expression - 3:06
```swift
let bullet =
    isRoot && (count == 0 || !willExpand) ? ""
        : count == 0    ? "- "
        : maxDepth <= 0 ? "▹ " : "▿ "
```

now can use at top-level as well, avoiding the need for a closure

##  Familiar and readable chain of if statements - 3:19
```swift
let bullet =
    if isRoot && (count == 0 || !willExpand) { "" }
    else if count == 0 { "- " }
    else if maxDepth <= 0 { "▹ " }
    else { "▿ " }
```

##  Initializing a global variable or stored property - 3:30
```swift
let attributedName = AttributedString(markdown: displayName)
```

##  In 5.9, if statements can be an expression - 3:46
```swift
let attributedName = 
				if let displayName, !displayName.isEmpty {
            AttributedString(markdown: displayName)
        } else {
            "Untitled"
        }
```

## Result builders
* faster type checking
* improved code completion
* more accurate error messages

##  In Swift 5.7, errors may appear in a different place - 4:31
```swift
struct ContentView: View {
    enum Destination { case one, two }

    var body: some View {
        List {
            NavigationLink(value: .one) { // The issue actually occurs here
                Text("one")
            }
            NavigationLink(value: .two) {
                Text("two")
            }
        }.navigationDestination(for: Destination.self) {
            $0.view // Error occurs here in 5.7
        }
    }
}
```

##  In Swift 5.9, you now receive a more accurate compiler diagnostic - 4:47
```swift
struct ContentView: View {
    enum Destination { case one, two }

    var body: some View {
        List {
            NavigationLink(value: .one) { //In 5.9, Errors provide a more accurate diagnostic
                Text("one")
            }
            NavigationLink(value: .two) {
                Text("two")
            }
        }.navigationDestination(for: Destination.self) {
            $0.view // Error occurs here in 5.7
        }
    }
}
```


## Type inference
type inferred from elements.  Swift generics preserve type information, so that your code operates seamlessly on concrete types.

##  An API that takes a request type and evaluates it to produce a strongly typed value - 5:47
```swift
struct Request<Result> { ... }

struct RequestEvaluator {
    func evaluate<Result>(_ request: Request<Result>) -> Result
}

func evaluate(_ request: Request<Bool>) -> Bool {
    return RequestEvaluator().evaluate(request)
}
```

##  APIs that abstract over concrete types and varying number of arguments - 6:03
```swift
let value = RequestEvaluator().evaluate(request)

let (x, y) = RequestEvaluator().evaluate(r1, r2)

let (x, y, z) = RequestEvaluator().evaluate(r1, r2, r3)
```

##  Writing multiple overloads for the evaluate function - 6:35
```swift
func evaluate<Result>(_:) -> (Result)

func evaluate<R1, R2>(_:_:) -> (R1, R2)

func evaluate<R1, R2, R3>(_:_:_:) -> (R1, R2, R3)

func evaluate<R1, R2, R3, R4>(_:_:_:_:)-> (R1, R2, R3, R4)

func evaluate<R1, R2, R3, R4, R5>(_:_:_:_:_:) -> (R1, R2, R3, R4, R5)

func evaluate<R1, R2, R3, R4, R5, R6>(_:_:_:_:_:_:) -> (R1, R2, R3, R4, R5, R6)
```

This forces an artificial upper-bound on the arguments you pass, resulting in compiler errors if you pass too many.   

In swift 5.9, the generics system gets first-class support for argument length.  New language concept that can represent multiple type parameters.

##  Individual type parameter - 7:12
```swift
<each Result>
```


##  Collapsing the same set of overloads into one single evaluate function - 7:36
```swift
func evaluate<each Result>(_: repeat Request<each Result>) -> (repeat each Result)
```

returns: a single value, or a tuple containing each value

##  Calling updated evaluate function looks identical to calling an overload - 8:21
```swift
struct Request<Result> { ... }

struct RequestEvaluator {
    func evaluate<each Result>(_: repeat Request<each Result>) -> (repeat each Result)
}

let results = RequestEvaluator.evaluate(r1, r2, r3)
```

you don't need to know that your API is using them!

[[Generalize APIS with parameter packs]]

Clear expression through concise code.  Swift's advanced language features, enable beautiful APIs that make it easy to say what you mean.

Swift 5.9 takes this design approach to the next level, by providing macros.

## Macros

##  It isn't clear why an assert function fails - 10:01
```swift
assert(max(a, b) == c)
```


##  XCTest provides an assert-equal operation - 10:20
```swift
XCAssertEqual(max(a, b), c) //XCTAssertEqual failed: ("10") is not equal to ("17")
```

this takes two values separately so you can at least see the two values.  

##  Assert as a macro - 11:02
```swift
#assert(max(a, b) == c)
```

Distributed as packages.  This comes from `PowerAssert`, an open-source package on GitHub.

If you were to look inside.

 ##  Macros are distributed as packages - 11:42
```swift
import PowerAssert
#assert(max(a, b) == c)
```

##  Macro declaration for assert - 12:07
```swift
public macro assert(_ condition: Bool)
```

otherwise, it looks a lot like a function.  If this macro produces a value, that result type would be written with the usual error syntax.

Uses are typechecked against parameters.  

##  Uses are type checked against the parameters - 12:26
```swift
import PowerAssert
#assert(max(a, b)) //Type 'Int' cannot be a used as a boolean; test for '!= 0' instead
```

Most macros are provided as 'external' macros.

##  A macro definition - 12:52
```swift
public macro assert(_ condition: Bool) = #externalMacro(
    module: “PowerAssertPlugin”,
    type: “PowerAssertMacro"
)
```

defines in separate programs that act as comipler plugins.  The swift compiler passes the sourcecode for macro use into the plugin.  Plugin produces new sourcecode which is inserted into the source program.



##  Swift compiler passes the source code for the use of the macro - 13:11
```swift
#assert(a == b)
```

##  Compiler plugin produces new source code, which is integrated back into the Swift program - 13:14
```swift
PowerAssert.Assertion(
    "#assert(a == b)"
) {
    $0.capture(a, column: 8) == $0.capture(b, column: 13)
}
```

you wouldn't want to write the boilerplate yourself, but we do it for you.

We also have *roles*.

##  Macro declarations include roles - 13:33
```swift
// Freestanding macro roles

@freestanding(expression)
public macro assert(_ condition: Bool) = #externalMacro(
    module: “PowerAssertPlugin”,
    type: “PowerAssertMacro"
)
```

freestanding, uses the `#` syntasx.  Expression because it is used anywhere that can produce a value.

##  New Foundation Predicate APIs uses a `@freestanding(expression)` macro role - 13:53
```swift
let pred = #Predicate<Person> {
    $0.favoriteColor == .blue
}

let blueLovers = people.filter(pred)
```

Write predicates in a type-safe manner using closures.    macro is generic over a set of input types. Accepts a closure argument of values over the input types.

##  Example of a commonly used enum - 14:14
```swift
enum Path {
    case relative(String)
    case absolute(String)
}
```

Often i need to filter all absolute paths from a collection.

##  Checking a specific case, like when filtering all absolute paths - 15:01
```swift
let absPaths = paths.filter { $0.isAbsolute }
```


##  Write an `isAbsolute` check as a computer property... - 15:09
```swift
extension Path {
    var isAbsolute: Bool {
        if case .absolute = self { true }
        else { false }
    }
}
```



##  ...And another for `isRelative` - 15:12
```swift
extension Path {
    var isRelative: Bool {
        if case .relative = self { true }
        else { false }
    }
}
```

this is tedious.  Macros can generate this boilerplate for us.

##  Augmenting the enum with an attached macro - 15:17
```swift
@CaseDetection
enum Path {
    case relative(String)
    case absolute(String)
}

let absPaths = paths.filter { $0.isAbsolute }
```



##  Macro-expanded code is normal Swift code - 15:36
```swift
enum Path {
    case relative(String)
    case absolute(String)
  
    //Expanded @CaseDetection macro integrated into the program.
    var isAbsolute: Bool {
        if case .absolute = self { true }
        else { false }
    }

    var isRelative: Bool {
        if case .relative = self { true }
        else { false }
    }
}
```

compiler integrates this into your program.  Debug into it, copy it out, etc.

| foo                          | bar                                                                       |
| ---------------------------- | ------------------------------------------------------------------------- |
| `@attached(member)`          | Adds new declarations inside the type/extension it's applied to           |
| `@attached(peer)`            | Adds new declarations alongside the declaration it's applied to           |
| `@attached(accessor)`        | adds accessors to a property                                              |
| `@attached(memberAttribute)` | Adds attributes to the declarations in the type/extension it's applied to |
| `@attached(conformance) `     | adds conformances to to the type/extension it's applied to                                                                       |

Several of these can be composed together to achieve usefule ffects.  Ex, observation.

## Observation

One conforms to ObservableObject and marks properties `@Published` and uses the `ObservedObject` property wrapper in the view.  Missing a step can mean the UI doesn't update as expected.

##  Observation in SwiftUI prior to 5.9 - 16:57
```swift
// Observation in SwiftUI

final class Person: ObservableObject {
    @Published var name: String
    @Published var age: Int
    @Published var isFavorite: Bool
}

struct ContentView: View {
    @ObservedObject var person: Person
    
    var body: some View {
        Text("Hello, \(person.name)")
    }
}
```

##  Observation now - 17:25

attaching the macro provides observation for all stored properties.

```swift
// Observation in SwiftUI

@Observable final class Person {
    var name: String
    var age: Int
    var isFavorite: Bool
}

struct ContentView: View {
    var person: Person
    
    var body: some View {
        Text("Hello, \(person.name)")
    }
}
```

This is a composition of 3 macro roles.

Person clas gets augmented in 3 ways.
1.  member introduces new properties and methods
2. Member attribute role will add `@ObservationTracked` macro to properties
3. These expand to getters/setters
4. conformance role introduces conformance to `Observable`.

##  Observable macro works with 3 macro roles - 17:42
```swift
@attached(member, names: ...)
@attached(memberAttribute)
@attached(conformance)
public macro Observable() = #externalMacro(...).
```

##  Unexpanded macro - 17:52
```swift
@Observable final class Person {
var name: String
    var age: Int
    var isFavorite: Bool
}
```

##  Expanded member attribute role - 18:05
```swift
@Observable final class Person {
    var name: String
    var age: Int
    var isFavorite: Bool
  
		internal let _$observationRegistrar = ObservationRegistrar<Person>()
    internal func access<Member>(
        keyPath: KeyPath<Person, Member>
    ) {
        _$observationRegistrar.access(self, keyPath: keyPath)
    }
    internal func withMutation<Member, T>(
        keyPath: KeyPath<Person, Member>,
        _ mutation: () throws -> T
    ) rethrows -> T {
        try _$observationRegistrar.withMutation(of: self, keyPath: keyPath, mutation)
    }
}
```

##  Member attribute role adds `@ObservationTracked` to stored properties - 18:12
```swift
@Observable final class Person {
    @ObservationTracked var name: String
    @ObservationTracked var age: Int
    @ObservationTracked var isFavorite: Bool
  
		internal let _$observationRegistrar = ObservationRegistrar<Person>()
    internal func access<Member>(
        keyPath: KeyPath<Person, Member>
    ) {
        _$observationRegistrar.access(self, keyPath: keyPath)
    }
    internal func withMutation<Member, T>(
        keyPath: KeyPath<Person, Member>,
        _ mutation: () throws -> T
    ) rethrows -> T {
        try _$observationRegistrar.withMutation(of: self, keyPath: keyPath, mutation)
    }
}
```

##  The @ObservationTracked macro adds getters and setters to stored properties - 18:16
```swift
@Observable final class Person {
    @ObservationTracked var name: String { get { … } set { … } }
    @ObservationTracked var age: Int { get { … } set { … } }
    @ObservationTracked var isFavorite: Bool { get { … } set { … } }
  
		internal let _$observationRegistrar = ObservationRegistrar<Person>()
    internal func access<Member>(
        keyPath: KeyPath<Person, Member>
    ) {
        _$observationRegistrar.access(self, keyPath: keyPath)
    }
    internal func withMutation<Member, T>(
        keyPath: KeyPath<Person, Member>,
        _ mutation: () throws -> T
    ) rethrows -> T {
        try _$observationRegistrar.withMutation(of: self, keyPath: keyPath, mutation)
    }
}
```

##  All that Swift code is folded away in the @Observable macro - 18:33
```swift
@Observable final class Person {
    var name: String
    var age: Int
    var isFavorite: Bool
}
```
When you need to see how a macro expands.  Just right click, and see Expand Macro.

## macro summary
* enable expressive APIs and eliminate boilerplate
* Macros integrate seamlessly into the development experience
[[Expand on Swift macros]]
[[Write Swift macros]]

# Swift everywhere

Despite these high-level capabilites cofeve, we can achieve low memory footprint.  This scalability means we're able to push swift to more places than was possible with ObjC.  To low-level systems, where previously you might expect to have to use C/C++.

Bringing Swift's clearer code and critical safety guarantees, to more places.

We recently open-sourced a foundation rewrite.  A single shared implementation on both apple and non-apple platforms.  But it also makes rewriting large amounts of C/objc in swift.

As of sonoma/iOS 17, there are new swift-backed implementations of date, calendar, formatting, internationalization, attributedstring, JSON encoding, decoding.  Performance wins have been significant.  

Calendar calculations: 20% faster
Date formatting: 150% faster
JSON coding: 200-500% faster

Both reducing bridging cost.  But also by the new swift-based implementations being faster.


benchmark

| foo   | ventura | sonoma    |
| ----- | ------- | --------- |
| objc  | 48ms    | 45ms (7%) |
| swift | 51ms    | 42ms (+20%)      |

Sometimes you need fine-grained control.  new, opt-in capabilities to achieve this level of control.  These focus on ownership.

## Ownership


##  A wrapper for a file descriptor - 23:59
```swift
struct FileDescriptor {
    private var fd: CInt
  
    init(descriptor: CInt) { self.fd = descriptor }

    func write(buffer: [UInt8]) throws {
        let written = buffer.withUnsafeBufferPointer {
            Darwin.write(fd, $0.baseAddress, $0.count)
        }
        // ...
    }
  
    func close() {
        Darwin.close(fd)
    }
}
```



##  The same FileDescriptor wrapper as a class - 24:30
```swift
class FileDescriptor {
    private var fd: CInt
  
    init(descriptor: CInt) { self.fd = descriptor }

    func write(buffer: [UInt8]) throws {
        let written = buffer.withUnsafeBufferPointer {
            Darwin.write(fd, $0.baseAddress, $0.count)
        }
        // ...
    }
  
    func close() {
        Darwin.close(fd)
    }
    deinit {
        self.close(fd)
    }
}
```

downsides: memory allocation.  In some constrained systems contexts, this is a problem.

Also have reference semantics.  You can share this across threads, store unintentionally, etc.



##  Going back to the struct - 25:05
```swift
struct FileDescriptor {
    private var fd: CInt
  
    init(descriptor: CInt) { self.fd = descriptor }

    func write(buffer: [UInt8]) throws {
        let written = buffer.withUnsafeBufferPointer {
            Darwin.write(fd, $0.baseAddress, $0.count)
        }
        // ...
    }
  
    func close() {
        Darwin.close(fd)
    }
}
```

it behaves 'like' a reference type.  It holds an integer which references the true value.  Making a copy of this type can also lead to unintentional sharing of state across your app.  What you want is to suppress the ability to make a copy of this struct.



##  Using Copyable in the FileDescriptor struct - 26:06
```swift
struct FileDescriptor: ~Copyable {
    private var fd: CInt
  
    init(descriptor: CInt) { self.fd = descriptor }

    func write(buffer: [UInt8]) throws {
        let written = buffer.withUnsafeBufferPointer {
            Darwin.write(fd, $0.baseAddress, $0.count)
        }
        // ...
    }
  
    func close() {
        Darwin.close(fd)
    }
  
    deinit {
        Darwin.close(fd)
    }
}
```

Swift types are `Copyable` by default.  This is usually the right choice.  Sometimes, that implicit copy isn't what you want.  In particular, when making copies of a value can lead to correctness issues.  In swift 5.9, we can restrict this.

Non-copyable can be deinit.  Can also be used to solve the problem of calling close and then using other methods.  The close operation can be `consuming`.  Calling this method gives up ownership of the value to the method you call.

##  `close()` can also be marked as consuming - 26:35
```swift
struct FileDescriptor {
    private var fd: CInt
  
    init(descriptor: CInt) { self.fd = descriptor }

    func write(buffer: [UInt8]) throws {
        let written = buffer.withUnsafeBufferPointer {
            Darwin.write(fd, $0.baseAddress, $0.count)
        }
        // ...
    }
  
    consuming func close() {
        Darwin.close(fd)
    }
  
    deinit {
        Darwin.close(fd)
    }
}
```



##  When `close()` is called, it must be the final use - 26:53
```swift
let file = FileDescriptor(fd: descriptor)
file.write(buffer: data)
file.close()
```

* By default, methods borrow their arguments.
* if you close the file first, and then attempt to call another method, you'll get `use after consume` error.

##  Compiler errors instead of runtime failures - 27:20
```swift
let file = FileDescriptor(fd: descriptor)
file.close() // Compiler will indicate where the consuming use is
file.write(buffer: data) // Compiler error: 'file' used after consuming
```

Non-copyable types are a powerful new feature for systems-level programming.  They're still at an early point.  Later versions will expand in generic code.  If you're interested in following along, see the swift forums.

## C++ interop
New build setting "C++ and objective-C interoperability".  


##  Using C++ from Swift - 28:52
```swift
// Person.h
struct Person {
    Person(const Person &);
    Person(Person &&);
    Person &operator=(const Person &);
    Person &operator=(Person &&);
    ~Person();
  
    std::string name;
    unsigned getAge() const;
};
std::vector<Person> everyone();

// Client.swift
func greetAdults() {
    for person in everyone().filter { $0.getAge() >= 18 } {
        print("Hello, \(person.name)!")
    }
}
```

C++ containers are accessible as swift collections.  We can write straightfowrard swift code that makes direct use of C++ functions and types.



##  Using Swift from C++ - 29:51
```swift
// Geometry.swift
struct LabeledPoint {
    var x = 0.0, y = 0.0
    var label: String = “origin”
    mutating func moveBy(x deltaX: Double, y deltaY: Double) { … }
    var magnitude: Double { … }
}

// C++ client
#include <Geometry-Swift.h>

void test() {
    Point origin = Point()
    Point unit = Point::init(1.0, 1.0, “unit”)
    unit.moveBy(2, -2)
    std::cout << unit.label << “ moved to “ << unit.magnitude() << std::endl;
}
```

Swift compiler will produce a generated header that contains a c++ view on the swift APIs.  Don't need to restrict yourself to `@objc`.  C++ can directly use most swift types and their full APIs.  Properties, methods, initializers, etc.

* use c++ apis directly from swift
* expose most swift APIs directly to C++
* join us on the forums!


##  An actor that manages a database connection - 35:30
```swift
// Custom actor executors

actor MyConnection {
    private var database: UnsafeMutablePointer<sqlite3>
  
    init(filename: String) throws { … }
  
    func pruneOldEntries() { … }
    func fetchEntry<Entry>(named: String, type: Entry.Type) -> Entry? { … }
}

await connection.pruneOldEntries()
```

[[Mix Swift and C++]]

Language level interopability is really important.  Also have to build your code.  xcode/swift package manager can be just as big a barrier as rewriting a large amount of code.

We worked with CMake to improve swift support there!

Mix C++/Swift within a single target.  CMake will compile them separately and link all libraries/runtimes for both languages.

## Actors and concurrency
Abstract model which can be adapted to different environments and libraries
* tasks
	* sequential unit of work that can run anywhere
* actors
	* Mutually exclusive access to isolated state

## Tasks in different environments
* global concurrency pool determines scheduling
	* dispatch on apple platforms
	* single-threaded cooperative queue in restricted environments
* Checked continuations provide interoperability with callback-based systems

keep in mind
* actors can be implemented in different ways
	* atomics?  spinlocks?
* Custom actor executors enable customization


##  MyConnection with a serial dispatch queue and a custom executor - 35:58
```swift
actor MyConnection {
  private var database: UnsafeMutablePointer<sqlite3>
  private let queue: DispatchSerialQueue
  
  nonisolated var unownedExecutor: UnownedSerialExecutor { queue.asUnownedSerialExecutor() }

  init(filename: String, queue: DispatchSerialQueue) throws { … }
  
  func pruneOldEntries() { … }
  func fetchEntry<Entry>(named: String, type: Entry.Type) -> Entry? { … }
}

await connection.pruneOldEntries()
```

Swift ensures mutual-exclusive access to the actor.  However, what if you need more control?  ex, what if you want to use a specific dispatch queue?  Because taht queue is sahred with other code that hasn't adopted actors.  Here we have added a serial dispatch queue to our actor, and an implementation of the `unownedExecutor` property.

With this change, all our actor synchronization happens on the queue. awaits outside the actor will dispatch_async onto the corresponding queue.

DispatchQueue conforms to

##  Dispatch queues conform to SerialExecutor protocol - 36:44
```swift
// Executor protocols

protocol Executor: AnyObject, Sendable {
    func enqueue(_ job: consuming ExecutorJob)
}

protocol SerialExecutor: Executor {
    func asUnownedSerialExecutor() -> UnownedSerialExecutor
    func isSameExclusiveExecutionContext(other executor: Self) -> Bool
}

extension DispatchSerialQueue: SerialExecutor { … }
```

You can provide your own synchronization mechanism.

* tasks and actors form the abstract model for Swift concurrency
* abstract model can be adapted to different execution environments
* abstract model provides customization points for tasks and actors

[[Swift concurrency Behind the scenes]]
[[Beyond the basics of structured concurrency]]

## case study: FoundationDB
* large, cross-platform C++ codebase
* heavy use of asynchronous programming based on C++ futures
* Distributed actor runtime providing deterministic simulation
* 


##  C++ implementation of FoundationDB's "master data" actor - 39:22
```swift
// C++ implementation of FoundationDB’s “master data” actor

ACTOR Future<Void> getVersion(Reference<MasterData> self, GetCommitVersionRequest req) {
  	state std::map<UID, CommitProxyVersionReplies>::iterator proxyItr = self->lastCommitProxyVersionReplies.find(req.requestingProxy);
  	++self->getCommitVersionRequests;

  	if (proxyItr == self->lastCommitProxyVersionReplies.end()) {
      	req.reply.send(Never());
    	  return Void();
  	}
  	wait(proxyItr->second.latestRequestNum.whenAtLeast(req.requestNum - 1));
  
  	auto itr = proxyItr->second.replies.find(req.requestNum);
  	if (itr != proxyItr->second.replies.end()) {
    		req.reply.send(itr->second);
    		return Void();
  	}

  	// ...
}
```

key aspects:
1.  C++ doesn't have async/await, so FoundationDB has their own preprocessor-like approach
2. like many codebases, they've implemented their own future type.
3. paired with explicit messaging to send responses to the requests.  Pairing of sending reply and returning.
4. Own reference-counted smart pointers.


##  Swift implementation of FoundationDB's "master data" actor - 40:18
```swift
// Swift implementation of FoundationDB’s “master data
actorfunc getVersion(
    myself: MasterData, req: GetCommitVersionRequest
) async -> GetCommitVersionReply? {
    myself.getCommitVersionRequests += 1

    guard let lastVersionReplies = lastCommitProxyVersionReplies[req.requestingProxy] else {
        return nil
    }

    // ...
    var latestRequestNum = try await lastVersionReplies.latestRequestNum
          .atLeast(VersionMetricHandle.ValueType(req.requestNum - UInt64(1)))

    if let lastReply = lastVersionReplies.replies[req.requestNum] {
        return lastReply
    }
}
```

# Wrap up
* create more expressive APIs
* Tune low-level performance
* Interoperate with existing c++ codebases
