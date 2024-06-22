Edit, debug, commit, repeat. Explore the suite of tools in Xcode that help you iterate quickly when developing apps. Discover tips and tricks to help optimize and boost your development workflow.

Projects slow down over time.

# Edit
## Find the right content

Type file's name in bottom left bar.
Specialized filters
Filter by git status

Xcode's find navigator -> cmd-shift-F.
Search across entire project.  

cmd to disclose/undisclose all sibling outlines.

Can delete items from the current query.

Press Cmd-E with any text selected and it will go into the find field with clipboard text unmolested.

## Move between files
Xcode has 2 types of tabs.
Permanent tabs -> edited code
implicit tab -> for files I've only passed through.  Disappears when I exit the file.  Italicized title.
Choose 'keep open' from the context menu or double click on the tab.

Option-clicking the close button of one tab.

Type while jump bar is open to filter the menu.

Cmd-Shift-J -> find jump bar item in navigator.
Option-drag in navigator to copy files.

right click, hold option, "new file with contents from clipboard".

Paste content directly into navigator!



### Warning and error annotations - 10:26
```swift
#warning("This is a warning annotation")
#error("This is an error annotation")
```

Consider bookmarks instead.
### Mark comments - 10:58
```swift
// MARK: This is a section title
```



## Leverage shortcuts

cmd-shift-O.
cmd-shift-A.  Do a search of xcode's commands with natural language.

Jump to definition (cmd-click).
option-click (show quick help)
edit all in scope (cmd-ctrl-E).  Rename a symbol and all its occurrences in the current file
Show callers.
ctrl-M: reformat to many lines
Double click on brace to jump to the other
Option->arrow to move by word.  Control->arrow to move by subword.  Cmd->left right helps move lines.

ctrl-shift-click to insert multiple cursors

### Placeholder - 14:09
```swift
<#placeholder#>
```

option->enter with completions.  Also option->tab.

Editor->Vim mode.

Basic emacs commands

| command | action            |
| ------- | ----------------- |
| \^A     | beginning of line |
| \^E     | end of line       |
| \^P     | Previous line     |
| \^N     | next line         |

## get the most out of git

"Show last change for line".

Preview upcoming commit in changes navigator.  

# Debug

## setting breakpoints
### showStarView() - 17:30
```swift
showStarView()
```

Use two breakpoints in tandem, disable/re-enable the busy breakpoint based on the low traffic breakpoint.
### Breakpoint #1 - 17:51
```swift
let task = URLSession.shared.dataTask(with: cloudURL, completionHandler: handleUpdatesFromCloud)
```

### Breakpoint #2 - 17:53
```swift
videos = loadVideosFromCloud()
```

Stop at throw instead of catch.
### Swift error breakpoint - 18:17
```swift
let url = try! getVideoResourceFilePath()
```



### Swift error throw - 18:34
```swift
throw URLLoadError.fileNotFound
```

### Conditional breakpoint - 18:59
```swift
cloudURL.scheme == "https"
```


### Print statement in conditional breakpoint - 19:18
```swift
p "Username is \(cloudURL.user())"
```


## using the console

### guard clause - 19:44
```swift
guard cloudURLs.allSatisfy({ $0. scheme == "https" }), session.configuration.networkServiceType == .video else { return }
```

### p session - 19:56
```swift
p session
```

### p first part of guard clause - 19:58
```swift
cloudURLs.allSatisfy({ $0. scheme == "https" })
```

### p second part of guard clause - 20:02
```swift
// Insert code snippet.
```

Did you know you can drag the pc around.

Deactivate all breakpoints.

### Random star rating - 20:11
```swift
var starRating: Int { let randomStarRating = Int.random(n: 1..<5) return randomStarRating }
```

Cmd-ctrl-R -> run without building.

see backtraces in the editor.  Debug bar, next to view/memory debuggers.


### Converting starRatingPercentage to Int - 21:16
```swift
var starRating: Int { return Int((starRatingPercentage * 5).rounded()) }
```

`#fileID`, `#function`.  See docs.


### print statements for debugging - 21:46
```swift
var releaseDate: Date { print("🎬 Entering func \(#function) in \(#fileID)...") let currentDate = Date() let gregorianCal = Calendar(identifier: .gregorian) var components = DateComponents() components.year = releaseYear print("\(#fileID)@\(#line) \(#function): 📅 releaseYear is \(releaseYear)") if releaseYear == gregorianCal.component(.year, from: currentDate) { components.month = Int(releaseMonth) isNewRelease = true print("\(#fileID)@\(#line) \(#function): 🆕 this is a new release!") } if releaseYear < 2000 { isClassicMovie = true print("\(#fileID)@\(#line) \(#function): 🎻 this one is a classic!") } let calendar = Calendar(identifier: .gregorian) return calendar.date(from: components)! }
```

Consider switching to os_log.
### os_log statements for debugging - 22:09
```swift
var releaseDate: Date { os_log(.debug, "🎬 Entering func \(#function) in \(#file)...") let currentDate = Date() let gregorianCal = Calendar(identifier: .gregorian) var components = DateComponents() components.year = releaseYear os_log(.info, "📅 releaseYear is \(releaseYear)") if releaseYear == gregorianCal.component(.year, from: currentDate) { components.month = Int(releaseMonth) isNewRelease = true os_log(.info, "🆕 this is a new release!") } if releaseYear < 2000 { isClassicMovie = true os_log(.info, "🎻 this one is a classic!") } let calendar = Calendar(identifier: .gregorian) return calendar.date(from: components)! }
```

Jump to source logging location

[[Run, Break, Inspect Explore effective debugging in LLDB]]

[[Debug with structured logging]]

[[Debug swift debugging with LLDB]]

# Test

Testing catches bugs.

Cmd-U.  

## Test navigator

re-run test.  Cmd-ctrl-option-G.

cmd-ctrl-U -> test without building

check - passed
x - failed
grey x -> expected failure
grey icon with arrow -> skipped
mixed green - some pass, some fail/skip
mixed red - > some tests failed


### Sample unit tests - 23:19
```swift
import Testing @testable 
import Destination_Video 
struct DestinationVideo_UnitTests { 
								   private var library = VideoLibrary() 
								   // Make sure starRating is returning a percentage 
								   @Test func testStarRating() async throws {
								    for video in library.videos {
								     #expect(video.info.starRating > 0) 
								     #expect(video.info.starRating <= 5)
								      } 
								      } 
								      // Make sure the library loads data from the json file 
								      @Test func testLibraryLoaded() async throws { 
								      #expect(library.videos.count > 1) 
								      } 
								      }
```

### Sample UI tests - 24:15
```swift
import XCTest final class Destination_VideoUITests: XCTestCase { private var app: XCUIApplication! @MainActor override func setUpWithError() throws { // UI tests must launch the application that they test. app = XCUIApplication() app.launch() // In UI tests it is usually best to stop immediately when a failure occurs. continueAfterFailure = false } @MainActor func testABeach() throws { // Tap the button to load the detail view for the "A Beach" video let aBeachButton = app.buttons["A Beach"].firstMatch aBeachButton.tap() // Make sure it has a Play Video button after going to that view let playButton = app.buttons["Play Video"] XCTAssert(playButton.exists) // Make sure the star rating for this video contains 4 stars to avoid issue we saw previously where it was only a single star because starRating was incorrectly a percentage instead of an Int let theRatingView = app.staticTexts["TheRating"] XCTAssert(theRatingView.label.contains("⭐️⭐️⭐️⭐️⭐️")) } @MainActor func testMainView() throws { // We should have at least 10 buttons for the various videos let buttons = app.buttons XCTAssert(buttons.count >= 10) // Check that the most popular videos have buttons for them for expectedVideo in ["By the Lake", "Camping in the Woods", "Ocean Breeze"] { XCTAssert(app.buttons[expectedVideo].exists) } } @MainActor func testLaunchPerformance() throws { if #available(macOS 10.15, iOS 13.0, tvOS 13.0, watchOS 7.0, *) { // This measures how long it takes to launch your application. measure(metrics: [XCTApplicationLaunchMetric()]) { XCUIApplication().launch() } } } }
```

### Swift Testing tags - 24:19
```swift
@Test(.tags(.stars)) 
func testStarRating() async throws {
									for video in library.videos {
									 #expect(video.info.starRating > 0) 
									 #expect(video.info.starRating <= 5) 
									 } 
									 } @Test(.tags(.library)) func testLibraryLoaded() async throws { #expect(library.videos.count > 1) } extension Tag { @Tag static var stars: Tag @Tag static var library: Tag }
```

## running tests


Can run repeatedly until failure.

### Running xcodebuild test from the command line - 26:35
```swift
xcodebuild test -scheme DestinationVideo
xcodebuild test -scheme DestinationVideo -testPlan TestAllTheThings
xcodebuild test -scheme DestinationVideo -testPlan TestAllTheThings -only-testing "Destination VideoUITests/testABeach"
```

Xcode Cloud.  25 hours per month.

[[Create practical workflows in Xcode Cloud]]
[[Get the most out of Xcode Cloud]]
[[Author fast and reliable tests for Xcode Cloud]]

## test plans



## code coverage
Determine how much of your codebase is executed when running tests.

cmd-9.  File-by-file how much coverage you have.
### Missing Code Coverage - 29:03
```swift
func toggleUpNextState(for video: Video) { if !upNext.contains(video) { // Insert the video at the beginning of the list. upNext.insert(video, at: 0) } else { // Remove the entry with the matching identifier. upNext.removeAll(where: { $0.id == video.id }) } // Persist the Up Next state to disk. saveUpNext() }
```

### Code Coverage executed 5 times - 29:19
```swift
init() { // Load all videos available in the library. videos = loadVideos() // The first time the app launches, set the last three videos as the default Up Next items. upNext = loadUpNextVideos(default: Array(videos.suffix(3))) }
```
## test report

See docs
[[Meet Swift Testing]]
# Distribute
## TestFlight
up to 10k users.  All platforms, etc.

[[Get started with TestFlight]]
"Including notes for testers with a beta release of your app" docs
[[Simplify distribution in Xcode and Xcode Cloud]]

## Archiving


## Organizer

cmd option shift O.

feedback.





# Resources
* https://developer.apple.com/documentation/Xcode/including-notes-for-testers-with-a-beta-release-of-your-app
* https://developer.apple.com/documentation/Xcode/including-notes-for-testers-with-a-beta-release-of-your-app
* https://developer.apple.com/documentation/Updates/Xcode
