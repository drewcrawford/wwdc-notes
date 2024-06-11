Join us for an update on Swift. We'll briefly go through a history of Swift over the past decade, and show you how the community has grown through workgroups, expanded the package ecosystem, and increased platform support. We'll introduce you to a new language mode that achieves data-race safety by default, and a language subset that lets you run Swift on highly constrained systems. We'll also explore some language updates including noncopyable types, typed throws, and improved C++ interoperability.

### Swift Build - 9:15
```swift
swift build
```

### Inspecting the build output - 9:20
```swift
file .build/debug/CatService
```

### Run the REST API service - 9:24
```swift
.build/debug/CatService
```

### Make a request to REST API service - 9:30
```swift
curl localhost:8080/api/emoji
```

### Install the Fully static Linux SDK for Swift - 9:45
```swift
swift sdk install ~/preview-static-swift-linux-0.0.1.tar.gz
```

### Swift build command flag to cross compile - 10:18
```swift
swift build --swift-sdk aarch64-swift-linux-musl
```

### Inspecting the build output - 10:30
```swift
file .build/debug/CatService
```

### Copy the service over to our Linux server and run it - 10:35
```swift
scp .build/debug/CatService demo-linux-host:~/CatService ./CatService
```

### Make a request to REST API service from macOS to Linux - 10:45
```swift
curl demo-linux-host:8080/api/emoji
```

### Swift Testing - Declare a test function - 13:50
```swift
// Swift Testing import Testing @Test func rating() { let video = Video(id: 2, name: "Mystery Creek") #expect(video.rating == "⭐️⭐️⭐️⭐️") }
```

### Swift Testing - Customize a test’s name - 13:55
```swift
// Swift Testing import Testing @Test("Recognized rating") func rating() { let video = Video(id: 2, name: "Mystery Creek") #expect(video.rating == "⭐️⭐️⭐️⭐️") }
```

### Swift Testing - Organize test function with tags - 14:13
```swift
// Swift Testing import Testing @Test("Recognized rating", .tags(.critical)) func rating() { let video = Video(id: 2, name: "Mystery Creek") #expect(video.rating == "⭐️⭐️⭐️⭐️") }
```

### Swift Testing - Parameterize test with arguments - 14:19
```swift
// Swift Testing import Testing @Test("Recognized rating", .tags(.critical), arguments: [ (1, "A Beach", "⭐️⭐️⭐️⭐️⭐️"), (2, "Mystery Creek", "⭐️⭐️⭐️⭐️"), ]) func rating(videoId: Int, videoName: String, expectedRating: String) { let video = Video(id: videoId, name: videoName) #expect(video.rating == expectedRating) }
```

### Noncopyable types - 17:50
```swift
struct File: ~Copyable { private let fd: CInt init(descriptor: CInt) { self.fd = descriptor } func write(buffer: [UInt8]) { // ... } deinit { close(fd) } }
```

### Noncopyable types - 18:12
```swift
guard let fd = open(name) else { return } let file = File(descriptor: fd) file.write(buffer: data)
```

### Noncopyable types - 18:42
```swift
struct File: ~Copyable { private let fd: CInt init?(name: String) { guard let fd = open(name) else { return nil } self.fd = fd } func write(buffer: [UInt8]) { // ... } deinit { close(fd) } }
```

### C++ Interoperability - 22:29
```swift
struct Person { Person(const Person&) = delete; Person(Person &&) = default; // ... };
```

### C++ Interoperability - 22:34
```swift
struct Developer: ~Copyable { let person: Person init(person: consuming Person) { self.person = person } } let person = Person() let developer = Developer(person: person)
```

### C++ Interoperability - 22:40
```swift
struct Developer: ~Copyable { let person: Person init(person: consuming Person) { self.person = person } } let person = Person() let developer = Developer(person: person) person.printInfo()
```

### Untyped throws - 23:43
```swift
enum IntegerParseError: Error { case nonDigitCharacter(String, index: String.Index) } func parse(string: String) throws -> Int { for index in string.indices { // ... throw IntegerParseError.nonDigitCharacter(string, index: index) } } do { let value = try parse(string: "1+234") } catch let error as IntegerParseError { // ... } catch { // error is 'any Error' }
```

### Typed throws - 24:19
```swift
enum IntegerParseError: Error { case nonDigitCharacter(String, index: String.Index) } func parse(string: String) throws(IntegerParseError) -> Int { for index in string.indices { // ... throw IntegerParseError.nonDigitCharacter(string, index: index) } } do { let value = try parse(string: "1+234") } catch { // error is 'IntegerParseError' }
```

### Typed throws - any and Never error types - 24:39
```swift
func parse(string: String) throws -> Int { //... } func parse(string: String) throws(any Error) -> Int { //... } func parse(string: String) -> Int { //... } func parse(string: String) throws(Never) -> Int { //... }
```

### Passing a NonSendable reference across actor isolation boundaries - 28:02
```swift
class Client { init(name: String, balance: Double) {} } actor ClientStore { static let shared = ClientStore() private var clients: [Client] = [] func addClient(_ client: Client) { clients.append(client) } } @MainActor func openAccount(name: String, balance: Double) async { let client = Client(name: name, balance: balance) await ClientStore.shared.addClient(client) }
```

### Atomic - 28:52
```swift
import Dispatch import Synchronization let counter = Atomic<Int>(0) DispatchQueue.concurrentPerform(iterations: 10) { _ in for _ in 0 ..< 1_000_000 { counter.wrappingAdd(1, ordering: .relaxed) } } print(counter.load(ordering: .relaxed))
```

### Mutex - 29:21
```swift
import Synchronization final class LockingResourceManager: Sendable { let cache = Mutex<[String: Resource]>([:]) func save(_ resource: Resource, as key: String) { cache.withLock { $0[key] = resource } } }
```

# Resources
* https://developer.apple.com/forums/topics/programming-languages-topic?cid=vf-a-0010
* https://www.swift.org/install/
* https://www.swift.org/migration/documentation/migrationguide/
* https://www.swift.org/blog/
* https://www.swift.org/community/
* https://forums.swift.org/
* https://docs.swift.org/swift-book/
