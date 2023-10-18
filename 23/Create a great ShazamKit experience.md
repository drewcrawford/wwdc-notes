 Discover how your app can offer a great audio matching experience with the latest updates to ShazamKit. We'll take you through matching features, updates to audio recognition, and interactions with the Shazam library. Learn tips and best practices for using ShazamKit in your audio apps. For more on ShazamKit, check out "Create custom catalogs at scale with ShazamKit" from WWDC22 as well as "Explore ShazamKit" and "Create custom audio experiences with ShazamKit" from WWDC21.

Shazam CLI
Time-restricted media items
frequency skewing - two similar bits of audio

[[Create custom catalogs at scale with ShazamKit]]
[[Explore ShazamKit]]
 
## Match flow
convert audio into a special format called signatures.

Pass stream of audio buffers into a shazamkit session
find a match in the shamzam catalog or a custom catalog
session returns a match object with medi atems representing metadata of the match

display in your app

generating a signature from a stream of audio buffers or using a signature file, stored on disk.
signatures are irreversible.  Not possible to resconstruct the original recording from a signature, protecting privacy

catlaog is a group of signatures with media items.  

matches occur when a query signature i snoisy such as music in a restaurant.

I suggest downloading the sample code project.  I'll be making use of the project throughout this video.
# Audio recognition
Using the microphone.

1.  Ask for microphone pemrission
2. start recording
3. Pass recorded audio buffer to shazamkit
4. handle result

demo app in sample project.  

ask siri to help me find a song.

info.plist used to request microphone access.  (usage description?)
On initialization, I have a method to configure and setup the audio engine.  I install a tap to receive pcm buffers and prepare the engine.  Also, I have a match method which is called when I tap on the button.

Notice how I have a lot of setup code to configure the audio engine, etc.  This can be challenging ot get right, especially if you aren't familiar with audio programming.  To make recording and matching easier, we have a new API - `SHManagedSession`
* automatically handles recording
* easy to set up and use
* add microphone usage description

###  Single match with SHManagedSession - 6:46
```swift
let managedSession = SHManagedSession()

let result = await managedSession.result()
      
switch result {
 case .match(let match):
        print("Match found. MediaItemsCount: \(match.mediaItems.count)")
 case .noMatch(_):
        print("No match found")
 case .error(_, _):
        print("An error occurred")
}
```



###  Multiple matches with SHManagedSession - 7:16
```swift
let managedSession = SHManagedSession()

// Continuously match 
for await result in managedSession.results {
          
 switch result {
  case .match(let match):
        print("Match found. MediaItemsCount: \(match.mediaItems.count)")
  case .noMatch(_):
        print("No match found")
  case .error(_, _):
        print("An error occurred")
 }
}
```

This ensures i can keep recording audio for long periods.

###  Stop SHManagedSession - 7:37
```swift
let managedSession = SHManagedSession()

// Cancel the session
managedSession.cancel()
```

cancels any currently-running match attempt and stops recording.  

converting SHSession to SHManagedSession - our new high level API.

Even more to talk about.

## Preparing SHManagedSession
prepare ahead of time?
* more responsive when matching
* pre-allocates necessary resources
* starts prerecording

match without preparing:
1.  result()
2. allocate resources
3. starts recording
4. returns a match

with preparing:
* pre-allocates resources
* starts pre-recording
* when you ask for a result, we return faster

###  ShazamKit Matcher with SHManagedSession - 8:02
```swift
import Foundation
import ShazamKit

struct MatchResult: Identifiable, Equatable {
    let id = UUID()
    let match: SHMatch?
}

@MainActor final class Matcher: ObservableObject {
    
    @Published var isMatching = false
    @Published var currentMatchResult: MatchResult?
    
    var currentMediaItem: SHMatchedMediaItem? {
        currentMatchResult?.match?.mediaItems.first
    }

    private let session: SHManagedSession
    
    init() {
        
        if let catalog = try? ResourcesProvider.catalog() {
            session = SHManagedSession(catalog: catalog)
        } else {
            session = SHManagedSession()
        }
    }
    
    func match() async {
        
        isMatching = true
        
        for await result in session.results {
            switch result {
            case .match(let match):
                Task { @MainActor in
                    self.currentMatchResult = MatchResult(match: match)
                }
            case .noMatch(_):
                print("No match")
                endSession()
            case .error(let error, _):
                print("Error \(error.localizedDescription)")
                endSession()
            }
            stopRecording()
        }
    }
    
    func stopRecording() {
        
        session.cancel()
    }
    
    func endSession() {
        
        // Reset result of any previous match.
        isMatching = false
        currentMatchResult = MatchResult(match: nil)
    }
}
```
###  Preparing SHManagedSession - 10:07
```swift
let managedSession = SHManagedSession()

await managedSession.prepare()

let result = await managedSession.result()
```

this is up to you, we call it on your behalf if necessary.

How do I track the current state of a managed session?

* idle
	* has completed a single match attempt
	* was cancelled
	* terminated async sequence of results
* prerecording
	* session has been prepared for a match attempt
	* prepare was called
	* precording has commenced
* matching
	* session is making at least one match attempt
	* calling `prepare` in this state will be ignored

###  SHManagedSession Idle State in SwiftUI - 11:39
```swift
struct MatchView: View {
    let session: SHManagedSession
    
    var body: some View {
        VStack {
             Text(session.state == .idle ? "Hear Music?" 
                : "Matching")
             if session.state == .matching {
                  ProgressView()
             } else {
                  Button {
                      // start match
                  } label: {
                      Text("Learn the Dance")
                  }
             }
        }       
}
```

###  SHManagedSession Matching State in SwiftUI - 12:25
```swift
struct MatchView: View {
    let session: SHManagedSession
    
    var body: some View {
        VStack {
             Text(session.state == .idle ? "Hear Music?" 
                : "Matching")
             if session.state == .matching {
                  ProgressView()
             } else {
                  Button {
                      // start match
                  } label: {
                      Text("Learn the Dance")
                  }
             }
        }    
    }
}
```

Since the state is currently idle, the button is displayed.  When I tap on the button, the state changes to matching and our UI refreshes.  Ex.

Whenever the state of the session changes, swiftui will refresh your views to respond to those changes without extra work.  Because `SHManagedSession` conforms to `Observable`

[[Discover observation in SwiftUI]]

SwiftUI can easily respond to any state changes of managed session.  See the related talk
# Shazam library

in 2021, we provided API to allow developers to add match results to the shazam library privided it has a shazam ID, ex in the catalog.  The added item is visible in control center and shazam app (if installed).  Synced across devices.  No permission required.  All songs saved in the library will be attributed to the app adding them.

over the years, usage of this API had drawbacks
* added item only visible in control center music recognition history
* manage storage of matched results in app

SHLibrary.  I recommend adopting this as it adds more extensive features vs SHMediaLibrary.

* add media items to shazam library
* read the media items you've added
* delete items
your app can only read and delete what it has added!

Items are specific to your app and do not represent the entire library.  Attempting to delete another app's media item will error.

```swift
func add(mediaItems: [SHMediaItem]) async throws {

    try await SHLibrary.default.addItems(mediaItems)
}
```

```swift
struct LibraryView: View {
    var body: some View {
        List(SHLibrary.default.items) { item in
            MediaItemView(item: item)
        }
    }
}
```

pass items into list initializer.  We conform to `Observable` so swiftui automatically reloads.



###  Reading with SHLibrary in a non-UI context - 16:00
```swift
// Determine a user’s most popular genre
   
let currentItems = await SHLibrary.default.items

let genres = currentItems.flatMap { $0.genres }

// count frequency of genres and get the highest
let mostPopularGenre = highestOccurringGenre(from: genres)
```

###  SHLibrary Remove - 16:25
```swift
func remove(mediaItems: [SHMediaItem]) async throws {
    
    try await SHLibrary.default.removeItems(mediaItems)
}
```

since I've added recognized songs, I can use new SHlibrary to read these songs.
###  RecentDancesView with SHLibrary read and delete implementation - 16:42
```swift
import SwiftUI
import ShazamKit

enum NavigationPath: Hashable {
    case nowPlayingView(videoURL: URL)
    case danceCompletionView
}

struct RecentDancesView: View {
    private enum ViewConstants {
        static let emptyStateImageName: String = "EmptyStateIcon"
        static let emptyStateTextTitle: String = "No Dances Yet?"
        static let emptyStateTextSubtitle: String = "Find some music to start learning"
        static let deleteSwipeViewOpacity: Double = 0.5
        static let matchingStateTextTopPadding: CGFloat = 24
        static let matchingStateTextBottomPadding: CGFloat = 16
        static let progressViewScaleEffect: CGFloat = 1.1
        static let progressViewBottomPadding: CGFloat = 12.0
        static let learnDanceButtonWidth: CGFloat = 250
        static let curvedTopSideRectangleHeight: CGFloat = 200
        static let listRowBottomInset: CGFloat = 30.0
        static let matchingStateText: String = "Get Ready..."
        static let notMatchingStateText: String = "Hear Music?"
        static let noMatchText: String = "No dance video for audio"
        static let navigationTitleText: String = "Recent Dances"
        static let learnDanceButtonText: String = "Learn the Dance"
        static let retryButtonText: String = "Try Again"
        static let cancelButtonText: String = "Cancel"
    }
    
    // MARK: Properties
    private var isListEmpty: Bool {
        SHLibrary.default.items.isEmpty
    }
    
    @State private var matchingState: String = ViewConstants.notMatchingStateText
    @State private var matchButtonText: String = ViewConstants.learnDanceButtonText
    @State private var canRetryMatchAttempt = false
    @State private var navigationPath: [NavigationPath] = []
    
    // MARK: Environment
    @EnvironmentObject private var matcher: Matcher
    @Environment(\.openURL) var openURL
    var body: some View {
        NavigationStack(path: $navigationPath) {
            ZStack(alignment: .bottom) {
                List(SHLibrary.default.items, id: \.self) { mediaItem in
                        RecentDanceRowView(mediaItem: mediaItem)
                            .onTapGesture(perform: {
                                guard let appleMusicURL = mediaItem.appleMusicURL else {
                                    return
                                }
                                openURL(appleMusicURL)
                            })
                            .swipeActions {
                                Button {
                                    Task { try? await SHLibrary.default.removeItems([mediaItem]) }
                                } label: {
                                    Image(systemName: "trash")
                                        .symbolRenderingMode(.hierarchical)
                                }
                                .tint(.appPrimary.opacity(0.5))
                            }
                }
                    .listStyle(.plain)
                    .overlay {
                        if isListEmpty {
                            ContentUnavailableView {
                                Label(ViewConstants.emptyStateTextTitle,
                                      image: ImageResource(name: ViewConstants.emptyStateImageName, bundle: Bundle.main))
                                    .font(.title)
                                    .foregroundStyle(Color.white)
                            } description: {
                                Text(ViewConstants.emptyStateTextSubtitle)
                                    .foregroundStyle(Color.white)
                            }
                        }
                    }
                    .safeAreaInset(edge: .bottom, spacing: ViewConstants.listRowBottomInset) {
                        ZStack(alignment: .top) {
                            CurvedTopSideRectangle()
                            VStack {
                                Text(matchingState)
                                    .font(.body)
                                    .foregroundStyle(.white)
                                    .padding(.top, ViewConstants.matchingStateTextTopPadding)
                                    .padding(.bottom, ViewConstants.matchingStateTextBottomPadding)
                                if matcher.isMatching {
                                    ProgressView()
                                        .progressViewStyle(.circular)
                                        .tint(.appPrimary)
                                        .scaleEffect(x: ViewConstants.progressViewScaleEffect, y: ViewConstants.progressViewScaleEffect)
                                        .padding(.bottom, ViewConstants.progressViewBottomPadding)
                                    Button(ViewConstants.cancelButtonText) {
                                        canRetryMatchAttempt = false
                                        matcher.stopRecording()
                                        matcher.endSession()
                                    }
                                    .foregroundStyle(Color.appPrimary)
                                    .font(.subheadline)
                                    .fontWeight(.semibold)
                                } else {
                                    Button {
                                        Task { await matcher.match() }
                                        matchingState = ViewConstants.matchingStateText
                                        canRetryMatchAttempt = true
                                    } label: {
                                        Text(matchButtonText)
                                            .foregroundStyle(.black)
                                            .font(.title3)
                                            .fontWeight(.heavy)
                                            .frame(maxWidth: .infinity)
                                    }
                                    .frame(width: ViewConstants.learnDanceButtonWidth)
                                    .padding()
                                    .background(Color.appPrimary)
                                    .clipShape(Capsule())
                                }
                            }
                        }
                        .edgesIgnoringSafeArea(.bottom)
                        .frame(height: ViewConstants.curvedTopSideRectangleHeight)
                    }
            }
            .background(Color.appSecondary)
            .navigationTitle(isListEmpty ? "" : ViewConstants.navigationTitleText)
            .preferredColorScheme(.dark)
            .toolbarColorScheme(.dark, for: .navigationBar)
            .navigationBarTitleDisplayMode(.large)
            .toolbarBackground(Color.appSecondary, for: .navigationBar)
            .frame(maxHeight: .infinity)
            .onChange(of: matcher.currentMatchResult, { _, result in
                
                guard navigationPath.isEmpty else {
                    print("Dance video already displayed")
                    return
                }
                
                guard let match = result?.match,
                      let url = ResourcesProvider.videoURL(forFilename: match.mediaItems.first?.videoTitle ?? "") else {
                    
                    matchingState = canRetryMatchAttempt ? ViewConstants.noMatchText : ViewConstants.notMatchingStateText
                    matchButtonText = canRetryMatchAttempt ? ViewConstants.retryButtonText : ViewConstants.learnDanceButtonText
                    return
                }
                
                canRetryMatchAttempt = false
                
                // Add the video playing view to the navigation stack.
                navigationPath.append(.nowPlayingView(videoURL: url))
            })
            .navigationDestination(for: NavigationPath.self, destination: { newNavigationPath in
                switch newNavigationPath {
                case .nowPlayingView(let videoURL):
                    NowPlayingView(navigationPath: $navigationPath, nowPlayingViewModel: NowPlayingViewModel(player: AVPlayer(url: videoURL)))
                case .danceCompletionView:
                    DanceCompletionView(navigationPath: $navigationPath)
                }
            })
            .onAppear {
                if AVAudioSession.sharedInstance().category != .ambient {
                    Task.detached { try? AVAudioSession.sharedInstance().setCategory(.ambient) }
                }
                matchingState = ViewConstants.notMatchingStateText
                matchButtonText = ViewConstants.learnDanceButtonText
            }
        }
    }
}
```

List of songs which the app has added to the shazam library.  I get this functionality for free and I don't need to maintain a database of matched songs.

implement swipe to delete, etc.

I've got the app open, I can swipe on an item on my iPhone and delete from my iPad.  Changes are synced.  This is amazing.
# Best practices

| SHManagedSession                        | SHSession                   |
| --------------------------------------- | --------------------------- |
| shazamkit handles recording             | handle recording yourself   |
| supports audio from microphone, airpods | microphone-only support     |
| no support for signature matching       | supports signature matching |
| shazamkit determines audio format       | choose your own audio formats                            |

## audio formats
previously we only allowed PCM buffers at specific sample rates.  With this release, SHSession now supports PCM with most formats.  Pass in these buffers and SHSession will handle conversion for you.

## custom catalogs
if you have two audio that sounds similar in a catalog, shazamkit can now pass in all matches if it matches multiple signatures.  Matches are returned sorted by best match quality.

as a tip, properly annotate the reference signatures that sound similar so you can distinguish between which result you want.

###  Filtering for specific media items - 20:23
```swift
func match(from televisionShowCatalog: SHCustomCatalog) async -> [SHMatchedMediaItem] {
        
        let managedSession = SHManagedSession(catalog: televisionShowCatalog)
        
        let result = await managedSession.result()
        
        if case .match(let match) = result {
            
            // filter for only media items related to a particular episode
            let filteredMediaItems = match.mediaItems.filter { $0.title == "Episode 2" }
            return filteredMediaItems
        }
        
        return []
}
```

say we have a TV show with the same intro music.  when matching the intro section, we return all the episode.  Filter through the media items for the episode I want.  

since I am using managed session, it can listen to airpods.  


# Resources
* https://developer.apple.com/documentation/shazamkit
