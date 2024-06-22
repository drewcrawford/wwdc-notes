Introducing Swift Testing: a new package for testing your code using Swift. Explore the building blocks of its powerful new API, discover how it can be applied in common testing workflows, and learn how it relates to XCTest and open source Swift.

Improve, maintain software quality over time.

A new open source package for testing your code with Swift.  Descriptive, organized tests
Actionable failures
scalable
designed for Swift

# Building blocks
###  Write your first @Test function - 1:54
```swift
import Testing
@Test func videoMetadata() {
  // ...
}
```

Test functions are annotated using `@Test`
Can be global functions or methods in a type
may be async or thorws
may be global-actor isolated



###  Validate an expected condition using #expect - 2:35
```swift
import Testing
@testable import DestinationVideo
@Test func videoMetadata() {
  let video = Video(fileName: "By the Lake.mov")
  let expectedMetadata = Metadata(duration: .seconds(90))
  #expect(video.metadata == expectedMetadata)
}
```

accepts ordinary expressions and operators
captures sourcecode and subexpressions that failed
we separately capture rhs and lhs.  We also see contents of array automatically.  Super handy.

Required expectations -> throw an error so we don't proceed.

Can use `#require` to unwrap an optional (and fail the test).

###  Fix a bug in the code being tested - 4:24
```swift
// In `Video.init(...)`
self.metadata = Metadata(forContentsOfUrl: url)
```

Name will be shown in the test navigator, etc.
###  Add a display name to a @Test function - 6:06
```swift
@Test("Check video metadata")
func videoMetadata() {
  let video = Video(fileName: "By the Lake.mov")
  let expectedMetadata = Metadata(duration: .seconds(90))
  #expect(video.metadata == expectedMetadata)
}
```

## traits
* add descriptive information about a test
* customize whether it runs

| trait                                             | usecase                                      |
| ------------------------------------------------- | -------------------------------------------- |
| `@Test("Custom name")`                            | customize display name                       |
| `@Test(.bug("example.com/issues/9999", "title"))` | reference issue from bug tracker             |
| `@Test(.tags(.critical))`                         | add custom tagset                            |
| `@Test(.enabled(if: Server.isOnline))`            | Specify runtime condition for test           |
| `@Test.disabled("Currently broken")`              | Unconditionally disable                      |
| `@Test(...) @available(macOS 15, *)`              | Limit to certain OS versions                 |
| `@Test(.timeLimit(.minutes(3)))`                  | Set maximum time limit for test              |
| `@Suite(.serialized)`                             | Run tests one at a time, no paralellization  |

###  Add a second @Test function - 6:58
```swift
@Test
func rating() async throws {
  let video = Video(fileName: "By the Lake.mov")
  #expect(video.contentRating == "G")
}
```

Suites group related test functions and suites
Annotated using `@Sutie`
Implicit for types containing `@Test` fucntions or suties
May have stored instance properties
Use init and deinit for setup and teardown logic respectively
Initialized once per instance `@Test` method

###  Organize @Test functions into a suite type - 7:18
```swift
struct VideoTests {
  @Test("Check video metadata")
  func videoMetadata() {
    let video = Video(fileName: "By the Lake.mov")
    let expectedMetadata = Metadata(duration: .seconds(90))
    #expect(video.metadata == expectedMetadata)
  }

  @Test
  func rating() async throws {
    let video = Video(fileName: "By the Lake.mov")
    #expect(video.contentRating == "G")
  }
}
```

Each one is called on its own instance of the suite so they don't share data.


###  Factor a common value into a stored property in the suite - 8:04
```swift
struct VideoTests {
  let video = Video(fileName: "By the Lake.mov")

  @Test("Check video metadata")
  func videoMetadata() {
    let expectedMetadata = Metadata(duration: .seconds(90))
    #expect(video.metadata == expectedMetadata)
  }

  @Test
  func rating() async throws {
    #expect(video.contentRating == "G")
  }
}
```


async/await and actor isolation.  

# Common workflows

Tests iwth conditions
Test with common characteristics
Tests with different arguments

Specify a runtime-evaluated condition for a test using `.enabled(if:)`.




###  Specify a runtime condition trait for a @Test function - 9:32
```swift
@Test(.enabled(if: AppFeatures.isCommentingEnabled))
func videoCommenting() {
  // ...
}
```

###  Unconditionally disable a @Test function - 9:49
```swift
@Test(.disabled("Due to a known crash"))
func example() {
  // ...
}
```

Preferable since it verifies test code still compiles.  

###  Include a bug trait with a URL along with other traits - 10:15
```swift
@Test(.disabled("Due to a known crash"), .bug("example.org/bugs/1234", "Program crashes at <symbol>"))
func example() {
  // ...
}
```

###  Conditionalize a test based on OS version - 10:33
```swift
@Test
@available(macOS 15, *)
func usesNewAPIs() {
  // ...
}
```
###  Prefer @available on @Test function instead of #available within the function - 10:42
```swift
// Insert code snippet.
```

## tests with common characteristics

assign tags to tests.

###  Add a tag to a @Test function - 11:22
```swift
@Test(.tags(.formatting))
func rating() async throws {
  #expect(video.contentRating == "G")
}
```

###  Add another data formatting-related test with the same tag - 11:48
```swift
@Test(.tags(.formatting))
func formattedDuration() async throws {
  let videoLibrary = try await VideoLibrary()
  let video = try #require(await videoLibrary.video(named: "By the Lake"))
  #expect(video.formattedDuration == "0m 19s")
}
```

Associate tests which have things in common to:
* run all tests with a specific tag
* filter by tag or see insights in test report

Shared among tests anywhere in a project
Tags can be local to a project or shared

Prefer tags over test names when including/excluding in a test plan
Use the most appropriate trait type: ex `.enabled(if:)` for runtime conditions.
[[Go further with Swift Testing]]



###  Group related tests into a sub-suite - 11:56
```swift
struct MetadataPresentation {
  let video = Video(fileName: "By the Lake.mov")

  @Test(.tags(.formatting))
  func rating() async throws {
    #expect(video.contentRating == "G")
  }

  @Test(.tags(.formatting))
  func formattedDuration() async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: "By the Lake"))
    #expect(video.formattedDuration == "0m 19s")
  }
}
```

###  Move common tags trait to @Suite attribute, so the suite's @Test functions will inherit the tag - 12:05
```swift
@Suite(.tags(.formatting))
struct MetadataPresentation {
  let video = Video(fileName: "By the Lake.mov")

  @Test
  func rating() async throws {
    #expect(video.contentRating == "G")
  }

  @Test
  func formattedDuration() async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: "By the Lake"))
    #expect(video.formattedDuration == "0m 19s")
  }
}
```


## tests with different arguments



###  Example of some repetitive tests which can be consolidated into a parameterized @Test function - 13:27
```swift
struct VideoContinentsTests {
  @Test
  func mentionsFor_A_Beach() async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: "A Beach"))
    #expect(!video.mentionedContinents.isEmpty)
    #expect(video.mentionedContinents.count <= 3)
  }

  @Test
  func mentionsFor_By_the_Lake() async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: "By the Lake"))
    #expect(!video.mentionedContinents.isEmpty)
    #expect(video.mentionedContinents.count <= 3)
  }

  @Test
  func mentionsFor_Camping_in_the_Woods() async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: "Camping in the Woods"))
    #expect(!video.mentionedContinents.isEmpty)
    #expect(video.mentionedContinents.count <= 3)
  }

  // ...and more, similar test functions
}
```

###  Refactor several similar tests into a parameterized @Test function - 14:07
```swift
struct VideoContinentsTests {
  @Test("Number of mentioned continents", arguments: [
    "A Beach",
    "By the Lake",
    "Camping in the Woods",
    "The Rolling Hills",
    "Ocean Breeze",
    "Patagonia Lake",
    "Scotland Coast",
    "China Paddy Field",
  ])
  func mentionedContinentCounts(videoName: String) async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require(await videoLibrary.video(named: videoName))
    #expect(!video.mentionedContinents.isEmpty)
    #expect(video.mentionedContinents.count <= 3)
  }
}
```


###  Example of using a for…in loop to repeat a test, instead of a parameterized @Test function (not recommended) - 16:29
```swift
// Using a for…in loop to repeat a test (not recommended)
@Test
func mentionedContinentCounts() async throws {
  let videoNames = [
    "A Beach",
    "By the Lake",
    "Camping in the Woods",
  ]
  let videoLibrary = try await VideoLibrary()
  for videoName in videoNames {
    let video = try #require(await videoLibrary.video(named: videoName))
    #expect(!video.mentionedContinents.isEmpty)
    #expect(video.mentionedContinents.count <= 3)
  }
}
```

parameterized testing advantages
* view details of each argument in results
* re-run individual arguments to debug
* run arguments in parallel

[[Go further with Swift Testing]]

Consider transforming these into parameterized tests.
###  Refactor a test using a for…in loop into a parameterized @Test function - 17:15
```swift
@Test(arguments: [
  "A Beach",
  "By the Lake",
  "Camping in the Woods",
])
func mentionedContinentCounts(videoName: String) async throws {
  let videoLibrary = try await VideoLibrary()
  let video = try #require(await videoLibrary.video(named: videoName))
  #expect(!video.mentionedContinents.isEmpty)
  #expect(video.mentionedContinents.count <= 3)
}
```


# Swift testing and XCTest

You might wonder how this compares!  Some similarities, some differences.

## test functions

| x                  | XCTest                                 | Swift Testing                                            |
| ------------------ | -------------------------------------- | -------------------------------------------------------- |
| Discovery          | name begins with 'test'                | `@Test`                                                  |
| Supported types    | instance methods                       | Instance methods, static/class methods, global functions |
| Supports traits    | no                                     | yes                                                      |
| Parallel execution | multi-process macOS and simulator only | In-process, supports devices                             |


## expectations
XCTAssert family vs expect/require.
Can pass in ordinary expressions and language operators.  ex, `==`, `>`, `!` etc.

In XCTests you assign `continueAfterFailure` to false.  In swift testing, any expectation can be made into required, by using `requires` instead of `expect`.  Choose which expectations should halt the test.
## suites

| x                | XCTest                                                              | Swift Testing                   |
| ---------------- | ------------------------------------------------------------------- | ------------------------------- |
| Discovery        | Subclass XCTestCase                                                 | `@Suite`                        |
| Before each test | `setUp` `setUpWithError throws` `setUp() async throws`              | `init() async throws`           |
| After each test  | `tearDown() async throws` `tearDownWithError() throws` `tearDown()` | `deinit` (reference type only?) |
| Subgroups        | unsupported                                                         | Via type nesting                |

Share a single unit test target, Swift testing can coexist with XCTests
Consolidate similar XCTests into a parameterized test
Migrate each XCTest class with only one test method to a global `@Test` function
word `test` is no longer necessary

Continue using XCTest for
* UI automation APIs (`XCUIApplication`)
* Performance testing APIs `XCTMetric`
* Can only be written in ObjC
Avoid calling XCTAssert from swift testing tests, or `#expect` from XCTests.

see migration docs.  
# Open source

we're on github.  

* apple platforms
* linux
* windows

Common codebase across all platforms.  

* swift package manager
* xcode 16
* vs code with swift extension


###  A newly-created Swift package with two simple @Test functions - 22:47
```swift
import Testing
@testable import MyLibrary

@Test
func example() throws {
  #expect("abc" == "abc")
}

@Test
func failingExample() throws {
  #expect(123 == 456)
}
```


###  Running all tests for the package from Terminal - 22:56
```bash
> swift test
```

* community feature proposal process
* discuss on swift forums
* contributions welcome!

# Wrap up
* improve quality with swift testing
* customize tests with traits
* contribute in open source

[[Go further with Swift Testing]]

# Resources
* https://developer.apple.com/documentation/Xcode/adding-tests-to-your-xcode-project
* https://developer.apple.com/documentation/Xcode/organizing-tests-to-improve-feedback
* https://developer.apple.com/documentation/Xcode/running-tests-and-interpreting-results
* https://developer.apple.com/documentation/Testing
* https://github.com/apple/swift-testing
* https://github.com/apple/swift-testing
