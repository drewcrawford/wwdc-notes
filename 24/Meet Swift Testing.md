Introducing Swift Testing: a new package for testing your code using Swift. Explore the building blocks of its powerful new API, discover how it can be applied in common testing workflows, and learn how it relates to XCTest and open source Swift.

###  Write your first @Test function - 1:54
```swift
import Testing
@Test func videoMetadata() {
  // ...
}
```

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

###  Fix a bug in the code being tested - 4:24
```swift
// In `Video.init(...)`
self.metadata = Metadata(forContentsOfUrl: url)
```

###  Add a display name to a @Test function - 6:06
```swift
@Test("Check video metadata")
func videoMetadata() {
  let video = Video(fileName: "By the Lake.mov")
  let expectedMetadata = Metadata(duration: .seconds(90))
  #expect(video.metadata == expectedMetadata)
}
```

###  Add a second @Test function - 6:58
```swift
@Test
func rating() async throws {
  let video = Video(fileName: "By the Lake.mov")
  #expect(video.contentRating == "G")
}
```

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
# Resources
* https://developer.apple.com/documentation/Xcode/adding-tests-to-your-xcode-project
* https://developer.apple.com/documentation/Xcode/organizing-tests-to-improve-feedback
* https://developer.apple.com/documentation/Xcode/running-tests-and-interpreting-results
* https://developer.apple.com/documentation/Testing
* https://github.com/apple/swift-testing
* https://github.com/apple/swift-testing
