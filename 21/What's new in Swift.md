#swift 

# Diversity
Foster an inclusive Swift community by elevating a wide variety of voices and making it easier to developers to... regardless of background.

swift.org/diversity

To make contributing to Swift more approachable, we announced the Swift Mentorship program.

# Update on Swift packages
Swift Package Index
Additional ways to find and access packages by integrating tooling support.

## Package collections in Xcode
Curated lists of Swift packages you can use from CLI and Xcode 13.  
Browse collections in xcode.

* Anyone can publish them (JSON file)
* Curated lists of packages for different use cases
	* Educator teaching a class
	* Curated set of packages that work together to accomplish a task
	* Enterprise approved packages

[[Discover and curate Swift packages using collections]]

When you try to import a module that cannot be found, Xcode will check if any configured collections provide that module and let you automatically start using the package.

All configuration is handled for you based on the information in the package collection.

Xcode comes prewired with the apple collection
swift.org/blog/package-collections

We launched 4 new packages.

## Swift Collections
A new open-source package of datastructures that complement stdlib.

github.com/apple/swift-collections

Deque - insert/removal from both ends
OrderedSet
OrderedDictionary


## Swift algorithms

Over 40 algorithms.  Combinations/permutations, iterating elements by 2/3s, or groups.
Find N smallest/largest, random, etc.

```swift
import Algorithms

let testAccounts = [ ... ]

for testGroup in testAccounts.uniquePermutations(ofCount: 0...) {
    try validate(testGroup)
}

let randomGroup = testAccounts.randomSample(count: 5)
```

[[Meet the Swift Algorithms and Collections packages]]

## Swift system
github.com/apple/swift-system

Idiomatic, low-level interfaces to system calls
Strong types, error handling, memory safety
Linux and Windows support

```swift
import System

let fd: FileDescriptor = try .open(
    "/tmp/a.txt", .writeOnly,
    options: [.create, .truncate], permissions: .ownerReadWrite)
try fd.closeAfter {
    try fd.writeAll("Hello, WWDC!\n".utf8)
}
```

```swift
import System

var path: FilePath = "/tmp/WWDC2021.txt"
print(path.lastComponent)         // "WWDC2021.txt"

print(path.extension)             // "txt"
path.extension = "pdf"            // path == "/tmp/WWDC2021.pdf"
path.extension = nil              // path == "/tmp/WWDC2021"
print(path.extension)             // nil

path.push("../foo/bar/./")        // path == "/tmp/wwdc2021/../foo/bar/."
path.lexicallyNormalize()         // path == "/tmp/foo/bar"
print(path.ends(with: "foo/bar")) // true!
```

## Numerics package
github.com/apple/swift-numerics
Float16 support on Apple silicon macs
Complex elementary functions

```swift
import Numerics

let z = Complex(0, Float16.pi) // πi
let w = Complex.exp(z)         // exp(πi) ≅ -1
```

## ArgumentParser
Fish shell completion scripts
Joined short options (`-Ddebug`)
Improved error messages

Adopted by SwiftPM.

## Update on Swift on server
* Static linking on Linux
* Improved JSON performance
* Enhanced AWS lambda runtime
	* 33% faster cold start time
	* 40% invocation time for APIGateway lambda

Now with async/await!

```swift
import AWSLambdaRuntime
import AWSLambdaEvents

@main
struct HelloWorld: LambdaHandler {
    typealias In = APIGatewayV2Request
    typealias Out = APIGatewayV2Response
    func handle(event: In, context: Lambda.Context) async throws -> Out {
        .init(statusCode: .ok, body: "Hello World")
    }
}
```

## Developer experience improvements
### Swift DocC

[[Meet DocC documentation in Xcode]]
[[Elevate your DocC documentation in Xcode]]
[[Host and automate your DocC documentation]]
[[Build interactive tutorials using DocC]]

Will be open-sourced later this year.  Will allow developers to more easily generate documentation on all platforms.

### Typechecker
Fewer "expression too complex to solve in reasonable time"
Performance of typechecking array literals

### Build improvements

* Faster builds when changing imported modules
* Faster startup time before launching compiles
* Fewer recompilations after changing an extension body

We now recompile less than a 10th 
buildtime decreased by 1/3rd

Modularize your project and change an imported module without a large penalty in build performance

First part of the compiler to be written in Swift.  Swift driver.

As of Xcode 13, is now the default.

## Memory management

Class instances use ARC to track references.  In most cases, this means that memory management just works and you don't need to think about it.

```swift
class Traveler {
    var destination: String
}

func test() {
    let traveler1 = Traveler(destination: "Unknown")
    // retain
    let traveler2 = traveler1
    // release
    traveler2.destination = "Big Sur"
    // release
    print("Done traveling")
}
```

This year, we introduced a new way to track references that allows the compiler to reduce retain/release ops.

Perf/codesign improvements related.

"Optimize Object Lifetimes" build setting enables this.

[[ARC in Swift: Basics and beyond]]

# Ergonomic improvements

* Multiple variadic parameters (SE-0284)
* Implicit member chains (SE-0287)
* Result builders (SE-0289)
* Property wrappers on parameters (SE-0293)
* Codable synthesis for associated value enums (SE-0295)
* Static member lookup in generic contexts (SE-0299)
* Interchangable use of CGFloat and Double (SE-0307)
* `#if` for postmix member expressions (SE-0308)

## Result builders SE-0289
* Originally designed for SwiftUI
* Flexible way to define complex object hierarchies
* Standardized and refined this year

[[Write a DSL in Swift using Result Builders]]

## Enum codable synthesis

```swift
enum Command {
    case load(key: String)
    case store(key: String, value: Int)
}
```

## Flexible static member lookup

```swift
enum Coffee {
    case regular
    case decaf
}

func brew(_ coffee: Coffee) { ... }

brew(.regular)
```
Now supported on protocols as well, by declaring static properties onto the protocol.
```swift
protocol Coffee { ... }
struct RegularCoffee: Coffee { }
struct Cappuccino: Coffee { }
extension Coffee where Self == Cappucino {
    static var cappucino: Cappucino { Cappucino() }
}

func brew<CoffeeType: Coffee>(_ coffee: CoffeeType) { ... }

brew(.cappucino.large) 
```

Improvements to Swift typechecker that allow it to reason more generally.  Allows library authors to build sophisticated generic data model.

## Property wrappers on parameters
```swift
@propertyWrapper
struct NonEmpty<Value: Collection> {
    init(wrappedValue: Value) {
        precondition(!wrappedValue.isEmpty)
        self.wrappedValue = wrappedValue
    }

    var wrappedValue: Value {
        willSet { precondition(!newValue.isEmpty) }
    }
}

func logIn(@NonEmpty _ username: String) {
    print("Logging in: \(username)")
}
```

## big demo

```swift
// Instead of writing this...

import SwiftUI

struct SettingsView: View {
    @State var settings: [Setting]

    private let padding = 10.0

    var body: some View {
        List(0 ..< settings.count) { index in
            #if os(macOS)
            Toggle(settings[index].displayName, isOn: $settings[index].isOn)
                .toggleStyle(CheckboxToggleStyle())
            #else
            Toggle(settings[index].displayName, isOn: $settings[index].isOn)
                .toggleStyle(SwitchToggleStyle())
            #endif
        }
        .padding(CGFloat(padding))
    }
}
```

```swift
// You can now write this.

import SwiftUI

struct SettingsView: View {
    @State var settings: [Setting]

    private let padding = 10.0

    var body: some View {
		//pass the projected binding directly into the list constructor.  Support for property wrpaper arguments lets us use `$setting`.
        List($settings) { $setting in
			//can now be factored out
            Toggle(setting.displayName, isOn: $setting.isOn)
              #if os(macOS)
			  //dot notation can be used here
              .toggleStyle(.checkbox)
              #else
			  	//dot notation can be used here

              .toggleStyle(.switch)
              #endif
        }
		//Swift compiler now transparently converts between CGFloat and Double.
        .padding(padding)
    }
}
```

[[20/what's new in swiftui]]

# Asynchronous and concurrent programming
```swift
// Instead of writing this...

func fetchImage(id: String, completion: (UIImage?, Error?) -> Void) {
    let request = self.imageURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) {
        data, urlResponse, error in
        if let error = error {
            completion(nil, error)
        } else if let httpResponse = urlResponse as? HTTPURLResponse,
                httpResponse.statusCode != 200 {
            completion(nil, MyTransferError())
        } else if let data = data, let image = UIImage(data: data) {
            completion(image, nil)
        } else {
            completion(nil, MyOtherError())
        }
    }
   task.resume()
}
```

Uses foundation URLSession to make a network call.
Datatask is async.  Result becomes available, etc.

Using closures to express asynchronous code results in an awkward order.

Completion handlers prevents us from using try/catch errors.

```swift
let (data, response) = try await URLSession.shared.data(for: request)
```
async function does not use any resources.  In particular, it's not blocking a thread.  So Swift can re-use the thread for other work.

This allows few threads to be shared among many async processes.
```swift
// You can now write this.

func fetchImage(id: String) async throws -> UIImage {
    let request = self.imageURLRequest(for: id)
    let (data, response) = try await URLSession.shared.data(for: request)
    if let httpResponse = response as? HTTPURLResponse,
       httpResponse.statusCode != 200 {
        throw TransferFailure()
    }
    guard let image = UIImage(data: data) else {
        throw ImageDecodingFailure()
    }
    return image
}
```

`async` decorates function, similar to the syntax of `throws`/`try`.

[[Meet asyncawait in Swift]]
[[Swift concurrency Behind the scenes]]

## Structured Concurrency

`async let`.  Initialization runs in parallel with other code.

bg/fg variables are initialized with async let, the runtime will (if necessary) suspend merge operation until values are ready.

Merge marked with `await` to indicate this.

```swift
func titleImage() async throws -> Image {
    async let background = renderBackground()
    async let foreground = renderForeground()
    let title = try renderTitle()
    return try await merge(background,
                           foreground,
                           title)
}
```

**Background tasks cannot outlive the function**.

**Function cannot return if background tasks are still running**.

If an error is thrown, runtime will wait for the task to be complete.

[[Explore structured concurrency in Swift]]

# Actors
```swift
actor Statistics {
    private var counter: Int = 0
    func increment() {
        counter += 1
    }
    func publish() async {
        await sendResults(counter)
    }
}

var statistics = Statistics()
await statistics.increment()
```

[[Protect mutable state with Swift actors]]

# Looking ahead to Swift 6
* Safe concurrency
* "even more efficient than it is today"

* Tell us about your experiences
* Try a compiler snapshot

