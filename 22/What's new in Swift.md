#swift 

# Community update
DocC and swift.org were opensourced.  
* Swift on Server
* Diversity in Swift
* Swift website
* C++ interop

358 mentees
41 membership pairs

mentorship in: language design, compielr development, technical writing, swift on server, software architecture, ttesting, ui programming, swift packatges, docc, c++ interop, swift website

year-round mentorship
swift.org/diversity

cross-paltform support
RPMS
* amazon linux 2
* centOS 7
*experimental*

dropped dependency on unicode support, replacing it with a native implementation.
statically linked stdlib.
* event-driven servers
* secure enclave

# Swift packages
TOFU
Trust On First Use

Command plugins
* generate docs
* reformat source code
* generate test reports

```swift
@main struct MyPlugin: CommandPlugin {

    func performCommand(context: PluginContext, arguments: [String]) throws {
        let process = try Process.run(doccExec, arguments: doccArgs)
        process.waitUntilExit()
    }

}
```

`swift package generate-documentation` on CLI.

Build tool plugins
Provide a scalable way for packages to deifne plugins that impelment additional build steps
* soruce code generation
* resource processing
differ from command plugins that execute directions at any time, can be granted permission to change files in a package.

In `Plugins` directory.  Treated as a swift executable.

```swift
import PackagePlugin

@main struct MyCoolPlugin: BuildToolPlugin {
    func createBuildCommands(context: TargetBuildContext) throws -> [Command] {

        let generatedSources = context.pluginWorkDirectory.appending("GeneratedSources")

        return [
            .buildCommand(
                displayName: "Running MyTool",
                executable: try context.tool(named: "mycooltool").path,
                arguments: ["create"],
                outputFilesDirectory: generatedSources)
        ]
    }
}
```

Provide extensibility in your packages.  [[Meet Swift Package plugins]] [[Create Swift Package plugins]]

## Module aliasing
two separate packages define a module with the same name.  Swift 5.7 introduces module disambig.  Featuer that allows you to rename modules from outside the package defining them.

```swift
let package = Package(
        name: "MyStunningApp",
        dependencies: [
            .package(url: "https://.../swift-metrics.git"),
            .package(url: "https://.../swift-log.git")
        ],
        products: [
            .executable(name: "MyStunningApp", targets: ["MyStunningApp"])
        ],
        targets: [
            .executableTarget(
                name: "MyStunningApp",
                dependencies: [
                    .product(name: "Logging", 
                             package: "swift-log"),
                    .product(name: "Metrics", 
                             package: "swift-metrics",
                             moduleAliases: ["Logging": "MetricsLogging"]),
  ])])
```


# Performance improvements
 * Integrated compiler
 * eager compilation
 * eager linking

[[Demystify parallelization in Xcode builds]]

* SwiftPM - 5% faster
* XCBuild - 25% faster

* Faster type checking of generics
* old implementation scaled exponentially
* 17s in swift 5.6, 760ms in 5.7
Runtime improvements:
* optimized protocol conformance checking
* Launch times being cut in half for some apps
* [[Improve app size and runtime performance]]

# Concurrency updates
* Back deployed to macOS 10.15, iOS 13, tvOS 13, watchOS 6

Extensions to the model
* Data race avoidance
* distributed actors
* async algorithms
## Data race avoidance
first consider memory safety
```swift
var numbers = [3, 2, 1]

numbers.removeAll(where: { number in
    number == numbers.count 
})
```

initially array's count is 3.  Once we've done that, the count will be 2.  
Swift prevents us from doing tihs because we have overlapping access.

Our goal is to do something simlar to data race safety.

```swift
var numbers = [3, 2, 1]

Task { numbers.append(0) } 

numbers.removeLast()
```
We refined the concurrency model to push us even further.

[[Eliminate data races using Swift Concurrency]]

Goal for swift 6: Thread safety.

New opt-in safety checks that identfy potential data races.
Strict concurrency checkign: minimal, targeted, complete.

## Distributed actors

makes developing distributed systems simpler.

```swift
distributed actor Player {
   
    var ai: PlayerBotAI?
    var gameState: GameState
    
    distributed func makeMove() -> GameMove {
        return ai.decideNextMove(given: &gameState)
    }
    
}
func endOfRound(players: [Player]) async throws {
    // Have each of the players make their move - some local, some remote
    for player in players {
        let move = try await player.makeMove()
    }
}
```
distributed actor can fail because of network errors.  Actor might throw for an error.

* makes distributed systems in Swift easier to write
* integrated networking solution
* integrated consensus protocol

[[Meet distributed actors in Swift]]

## Async algorithms package
* seamless integration with async/await
* home for time-based algorithms using AsyncSequence
* Support on apple platforms, linux, and windows

* zip creates tuples of values from several async sequences
* merge joins several async sequence itnoa  single stream
* debounce
* chunked
[[Meet Swift Async Algorithms]]

## optimizations
* actor prioritization
* priority-inversion avoidance

## Instruments
tasks, actors.  [[Visualize and optimize Swift concurrency]]

# Expressive Swift
When all you ahve is a hammer, everything looks like a nail

```swift
if let mailmapURL = mailmapURL {

    mailmapLines = try String(contentsOf: mailmapURL).split(separator: "\n")
    
}
```

VS
```swift
if let workingDirectoryMailmapURL {
  
    mailmapLines = try String(contentsOf: workingDirectoryMailmapURL).split(separator: "\n")

}

guard let workingDirectoryMailmapURL else { return }

mailmapLines = try String(contentsOf: workingDirectoryMailmapURL).split(separator: "\n")
```

Return inference now works for complex closures
```swift
let entries = mailmapLines.compactMap { line in

    try? parseLine(line)

}

func parseLine(_ line: Substring) throws -> MailmapEntry { … }
```


Permitted pointer conversions.  In C you can change signedness, etc.  But we don't allow that in Swift.  So imported C code has issues.

```swift
func removeDuplicates(from map: UnsafeMutablePointer<mailmap_t>) {
    var size = mailmap_get_size(map)
    size -= moveDuplicatesToEnd(map)
    withUnsafeMutablePointer(to: &size) { signedSizePtr in
        signedSizePtr.withMemoryRebound(to: UInt32.self, capacity: 1) { unsignedSizePtr in
            mailmap_truncate(map, unsignedSizePtr)
        }
    }
}
```

But in C, that pointer mismatch is legal.  So this isn't so dangerous.  Now, when calling C, we let you play by C rules.  Specifically can cast to other signess type, to UInt8, between raw (untyped, that is `void`) and typed pointer, etc.

## String processing
String parsing is hard.
```swift
func parseLine(_ line: Substring) throws -> MailmapEntry {
    func trim(_ str: Substring) -> Substring {
        String(str).trimmingCharacters(in: .whitespacesAndNewlines)[...]
    }

    let activeLine = trim(line[..<(line.firstIndex(of: "#") ?? line.endIndex)])
    guard let nameEnd = activeLine.firstIndex(of: "<"),
          let emailEnd = activeLine[nameEnd...].firstIndex(of: ">"),
          trim(activeLine[activeLine.index(after: emailEnd)...]).isEmpty else {
        throw MailmapError.badLine
    }

    let name = nameEnd == activeLine.startIndex ? nil : trim(activeLine[..<nameEnd])
    let email = activeLine[activeLine.index(after: nameEnd)..<emailEnd]

    return MailmapEntry(name: name, email: email)
}
```

People focus on indexes, but even if we improved them...
```swift
func parseLine(_ line: Substring) throws -> MailmapEntry {
    func trim(_ str: Substring) -> Substring {
        String(str).trimmingCharacters(in: .whitespacesAndNewlines)[...]
    }

    let activeLine = trim(line[..<(line.firstIndex(of: "#") ?? line.endIndex)])
    guard let nameEnd = activeLine.firstIndex(of: "<"),
          let emailEnd = activeLine[nameEnd...].firstIndex(of: ">"),
          trim(activeLine[(emailEnd + 1)...]).isEmpty else {
        throw MailmapError.badLine
    }

    let name = nameEnd == activeLine.startIndex ? nil : trim(activeLine[..<nameEnd])
    let email = activeLine[(nameEnd + 1)..<emailEnd]

    return MailmapEntry(name: name, email: email)
}
```

allegedly this is better, although I am not totally sold

```swift
func parseLine(_ line: Substring) throws -> MailmapEntry {
	/* wut */
    let regex = /\h*([^<#]+?)??\h*<([^>#]+)>\h*(?:#|\Z)/

    guard let match = line.prefixMatch(of: regex) else {
        throw MailmapError.badLine
    }

    return MailmapEntry(name: match.1, email: match.2)
}
```

> in a dense information-packed syntax

so let's do it with words.  Kind of like SwiftUI.

```swift
import RegexBuilder

let regex = Regex {
    ZeroOrMore(.horizontalWhitespace)
  
    Optionally {
        Capture(OneOrMore(.noneOf("<#")))
    }
        .repetitionBehavior(.reluctant)

    ZeroOrMore(.horizontalWhitespace)

    "<"
    Capture(OneOrMore(.noneOf(">#")))
    ">"

    ZeroOrMore(.horizontalWhitespace)
    ChoiceOf {
       "#"
       Anchor.endOfSubjectBeforeNewline
    }
}
```

reusable components
```swift
struct MailmapLine: RegexComponent {
    @RegexComponentBuilder
    var regex: Regex<(Substring, Substring?, Substring)> {
        ZeroOrMore(.horizontalWhitespace)

        Optionally {
            Capture(OneOrMore(.noneOf("<#")))
        }
            .repetitionBehavior(.reluctant)

        ZeroOrMore(.horizontalWhitespace)

        "<"
        Capture(OneOrMore(.noneOf(">#")))
        ">"

        ZeroOrMore(.horizontalWhitespace)
        ChoiceOf {
           "#"
            Anchor.endOfSubjectBeforeNewline
        }
    }
}
```

Insert literals directly into the builder

```swift
struct MailmapLine: RegexComponent {
    @RegexComponentBuilder
    var regex: Regex<(Substring, Substring?, Substring)> {
        ZeroOrMore(.horizontalWhitespace)

        Optionally {
            Capture(OneOrMore(.noneOf("<#")))
        }
            .repetitionBehavior(.reluctant)

        ZeroOrMore(.horizontalWhitespace)

        "<" 
        Capture(OneOrMore(.noneOf(">#")))
        ">" 

        ZeroOrMore(.horizontalWhitespace)
        /#|\Z/
   }
}
```

types (foundation) can integrate custom parsing logic

```swift
struct DatedMailmapLine: RegexComponent {
    @RegexComponentBuilder
    var regex: Regex<(Substring, Substring?, Substring, Date)> {
        ZeroOrMore(.horizontalWhitespace)

        Optionally {
            Capture(OneOrMore(.noneOf("<#")))
        }
            .repetitionBehavior(.reluctant)

        ZeroOrMore(.horizontalWhitespace)

        "<" 
        Capture(OneOrMore(.noneOf(">#")))
        ">" 

        ZeroOrMore(.horizontalWhitespace)

        Capture(.iso8601.year().month().day())

        ZeroOrMore(.horizontalWhitespace)
        /#|\Z/
   }
}
```

matching

```swift
func parseLine(_ line: Substring) throws -> MailmapEntry {

    let regex = /\h*([^<#]+?)??\h*<([^>#]+)>\h*(?:#|\Z)/
    // or let regex = MailmapLine()

    guard let match = line.prefixMatch(of: regex) else {
        throw MailmapError.badLine
    }

    return MailmapEntry(name: match.1, email: match.2)
}
```

new matching engine
* custom engine written ins wift
* literal dialect based on UTS #18 with extensions
* `.` matches a whole character not unicode scalar
* Available in macOS 13, iOS 16, tvOS 16, watchOS 9
* [[Meet Swift Regex]]
* [[Swift Regex Beyond the Basics]]

## Generic code clarity
```swift
/// Used in the commit list UI
struct HashedMailmap {
    var replacementNames: [String: String] = [:]
}

/// Used in the mailmap editor UI
struct OrderedMailmap {
    var entries: [MailmapEntry] = []
}

protocol Mailmap {
    mutating func addEntry(_ entry: MailmapEntry)
}

extension HashedMailmap: Mailmap { … }
extension OrderedMailmap: Mailmap { … }
```

PAT or generics?
```swift
func addEntries1<Map: Mailmap>(_ entries: Array<MailmapEntry>, to mailmap: inout Map) {
    for entry in entries {
        mailmap.addEntry(entry)
    }
}

func addEntries2(_ entries: Array<MailmapEntry>, to mailmap: inout Mailmap) {
    for entry in entries {
        mailmap.addEntry(entry)
    }
}
```

If you say `: Mailmap` or `some Mailmap` it means generic cosntraint.  But PAT is boxed.

Box => more space, more time to operate, and lacks capability.  Hard to figure out if you're using one.  In swift 5.7, when you're using PAT, you need to say `any`.  Not mandatory, but it is encouraged.

In simple cases, you can now pass PAT into a generic parameter.

Previously, we had restrictions on self and associated types.  Or conformed to ap rotocol that did, such as equatable.  In swift 5.7, this is gone.

`any Collection<MailmapEntry>`.  

## Primary associated types.
```swift
protocol Collection<Element>: Sequence {
    associatedtype Index: Comparable
    associatedtype Iterator: IteratorProtocol<Element>
    associatedtype SubSequence: Collection<Element>
                                    where SubSequence.Index == Index,
                                          SubSequence.SubSequence == SubSequence

    associatedtype Element
}
```

consider `Element` vs something like `Index`.  Index is more of an implementation detail.

When you have somethign like `Element`, you can put its name in angle brackets to make it a *primary associated type*.  

Now we have two PATS....

## Any redux
Dont' we already have `AnyCollection`?  It's a type-erasing wrapper that serves the same purpose as the any type.  But it's line after line of code.

whereas `any Collection` is the same thing for free.  This will stick around for compatibility and has a few features that we can't od yet, but see if you can reimplement with `any`.

or maybe just replace with typealiases.

## Improvements to any types
* any keyword
* pass to generic arguments
* self/associated types
* primary associated types

But they still have limitations:
* still can't use `==` operator between two `any` because they might not have the same type.
* **don't use them when generics work.**

But generics are exhausting.  Declare parameters, etc.  So let's introduce ~~impl~~ `some`.  Now supports primary associated types as well.

this pat thing is going to get old fast

[[Embrace Swift generics]]
[[Design protocol interfaces in Swift]]

but there are lots more changes!  Swift evolution etc

thanks for making 5.7 a great release!
swift.org/contributing

