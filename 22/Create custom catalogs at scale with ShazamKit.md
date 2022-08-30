In 2021, we introduced #shazamkit .  Custom catalog matching.

Streamline working with custom catalogs at scale.  In this session, I"m going to use existing shazamkit concepts such as signatures, catalogs, and media items.

Check out [[Explore ShazamKit]] and [[Create custom audio experiences with ShazamKit]]

Lets you convert audio into a special format that acn be matched, called *signatures*.  

Reference signature.  Stored together in a file called a custom catalog.

# Custom catalogs at scale
If you have a small amount of content to be matched,
1.  Record your audio in a format that ShazamKit accepts
2. use the signature generator to transform into a signature
3. Annotate with metadata
4. store in a custom catalog

Some of those steps can be daunting if you're not familiar with audio programming.  Sample rates, buffers, etc.  What happens when you ahve a vast amount of content?  10 seasons of a tv show?  etc.

Large amounts of content can quickly become unmanageable.  Write code to transform audio into signatures.  Load and associate media items, change your content, repeat the work, etc.  Big investment when you just want to match audio.

Sync content with shazam kit -> what to be shown and when.

First
## Demo
## How does it work?
```swift
/*
See LICENSE folder for this sample’s licensing information.

Abstract:
The model that is responsible for matching against the catalog and update the SwiftUI Views.
*/

import ShazamKit
import AVFAudio

struct MatchResult {
    var title: String?
    var equation: Equation?
    var episode: Episode?
    var answerRange: ClosedRange<Int>?
    
    var hasContent: Bool {
        equation != nil || title != nil || answerRange != nil
    }
}

class Matcher: NSObject, ObservableObject, SHSessionDelegate {
    @Published var matchResult: MatchResult?
    
    private var session: SHSession!
    private let audioEngine = AVAudioEngine()
    private var matchingTask: Task<Void, Never>? = nil
    
    func match(catalog: SHCustomCatalog) throws {
        
        session = SHSession(catalog: catalog)
        session.delegate = self
        
        let audioFormat = AVAudioFormat(standardFormatWithSampleRate: audioEngine.inputNode.outputFormat(forBus: 0).sampleRate,
                                        channels: 1)
        audioEngine.inputNode.installTap(onBus: 0, bufferSize: 2048, format: audioFormat) { [weak session] buffer, audioTime in
            session?.matchStreamingBuffer(buffer, at: audioTime)
        }
        
        try AVAudioSession.sharedInstance().setCategory(.record)
        AVAudioSession.sharedInstance().requestRecordPermission { [weak self] success in
            guard success, let self = self else { return }
            
            Task.detached {
                try? self.audioEngine.start()
            }
        }
        
        Task { @MainActor in
            for await case .match(let match) in session.results {
                self.matchResult = match.matchResult
            }
        }
    }
}

extension SHMatch {

    var matchResult: MatchResult {
        mediaItems.reduce(into: MatchResult()) { result, mediaItem in
            result.title = result.title ?? mediaItem.title
            result.episode = result.episode ?? mediaItem.episode
            result.equation = result.equation ?? mediaItem.equation
            result.answerRange = result.answerRange ?? mediaItem.answerRange
        }
    }
}
```
AsyncSequence on the session.  Returns an enum representing match, no match, error.  Only interested in matches so I restricted the loop to just that case.  I reduce the media items to the content that I need.

Actually not much more to see in the app, just SwiftUI views driven by the match result.  No complicated logic or timing code, syncs perfectly.  How does it sync so well?

Catalog.

Shazam CLI.  Ships as part of macOS 13, provides an easy to sync content.  Automates repetitive tasks.

CLI Accepts a simple, comma-separated file representing media items.  It describes everything that i need to sync my content.  Here's where I specified my titles, etc.  Custom JSON field that I've defined for the equation.  Headers map to media item properties.  Detail on the mapping runs the custom catalog create command.

It describes the relationship between CSV headers and the item properties.  
`shazam custom-catalog create --signature-asset File.shazamcignature --media-items example.csv ...`

That's a quick overview of how to create catalogs.  But oyu may want to script this.

Food map app has quite a few new episodes.  Add them all to this catalog.  Simple script that loops through episode folders and ocmbines into a custom catalog.

* Create => a signature from any media file that has an audio track.  Custom catalogs taht combine sigantures and media items
* display
* update
* remove
* export

Create a signature from an AVAsset.  `SHSigantureGenerator.signature(from:)`.  

Timed media item API.  Attaching at ime range to the media items make it easy to specify where it starts and ends.  Multiple ranges to target more than one portion of a siganture.  Suppose you have a media item that targets a chorus, can have a time range for each time i'ts sung.

ShazamKit delivers  a match callback synced with the time range, when it starts and ends.  This callback contians only media items at that point in time.

Rules.
Media items outside their time range are not returned
media items within their time range ordered by most recent
mediaitems with no time ranges will always be returned last and unordered
	store global information that applies to the whole reference signature.  ex, name of the episode.

If all your media items have time ranges and none are inscope, it will reutrn a media tiem with basic match information.  So you'll always get properties like predictedCurrentMatchOffset, frequency skew, etc.

```swift
// Restrict this media item to only describe the first 5 seconds

 let mediaItem = SHMediaItem(properties: [
            .title: "Title",
            .timeRanges:[0.0..<5.0]
        ])

 let timeRanges: [Range<TimeInterval>] = mediaItem.timeRanges
 ```

An array of swift ranges.  It can be read back using the timeRanges property.  For objc programmers, there's a new `SHRange` class that's a drop-in replacement.

# Tips and tricks
Avoid creating many small signatures for one piece of media.  A signature is a 1-1 mapping to the media that it represents.  For each pieve of audio, create one signature for the entire duration.

Match peaks.  Better accuracy, etc.  Issues with query signatures, overlapping multiple reference signatures.  Using timed media API, you can target synced content with individual areas.  No need to divide audio into multipel signatures.

What if we have a huge amount of content?

Tradeoff

| One catalog                       | Multiple catalogs                |
| --------------------------------- | -------------------------------- |
| detect multiple pieces of content | Must know the content in advance (to load the correct catalog) |
| large file size                   | smaller file size                |
| increased memory usage            | smaller memory usage                                 |

Our advice is to keep catalogs tightly focused.  Per music track, or whole album.  But not the whole discography.

Decide what to load at runtime.  With custom catalog `.add` API.  See if it helps with your usecase.
```swift
let parentCatalog = SHCustomCatalog()

parentCatalog.add(from: URL(fileURLWithPath: "/path/to/Episode1.shazamcatalog"))
parentCatalog.add(from: URL(fileURLWithPath: "/path/to/Episode2.shazamcatalog"))
parentCatalog.add(from: URL(fileURLWithPath: "/path/to/Episode3.shazamcatalog"))
```
## Frequency skew
Different tracks with sections that start the same
Reuse audio in different contexts
Skewing audio is raising or lowering frequencies in the recording.  You affect how the audio sounds, but if you do it by small amounts it can be noticed by shazamkit but not by average human.
```swift
func within(range: Range<Float>, for matchedMediaItem: SHMatchedMediaItem) -> Bool {
        
	range.contains(matchedMediaItem.frequencySkew)
}
```

Limits to how much you can do without being noticeable to the human ear.  Keeping skew within 5% should be safe.

Consider using ranges.  Only return items within the skew range.
```swift
// Restrict this media item to only describe the first 5 seconds

 let mediaItem = SHMediaItem(properties: [
            .title: “Frequency Skewed Audio”,
            .frequencySkewRanges:[0.01..<0.02]
        ])

 let frequencySkewRanges: [Range<Float>] = mediaItem.frequencySkewRanges
```

1.  Create reference signature
2. Media item and restrict it by skew
3. Place inside your custom catalog
4. Play back the audio, skewed by 3-4%
5. media item will be returned.
6. unskewed or outside the range will not return the item.

# Wrap up
* create one signature per asset
* Create signatures with `signatureFromAsset`
	* wide variety of media, no longer ahve to deal with low-level details
* Sync content with timed media items
* Automate workflows with shazam CLI


* https://developer.apple.com/forums/tags/wwdc2022-10028
* https://developer.apple.com/forums/create/question?&tag1=57030&tag2=473030
* https://developer.apple.com/documentation/shazamkit

