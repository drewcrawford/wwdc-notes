When using media ssets, you might like to do more than just play them.  Thumbnails, combine media into new comopsitions, or get information about your assets.

Require loading data.  With big files like video, might take awhile.

It can be easy to introduce latency by doing this on the main thread.  Keep your app responsive, async.

# New async API
Isn't instantaneous.  To generate an image, we need frame data.  For media on a remote server, that loading will be slower.  Important how you generate images.

Using a method that loads data synchronously like `copyCGImage` not recommended.

This year, we have `.image(at: time)`.  returns a tuple with the image and its actual time in the asset.  A few reasons why the actual time may vary from what you requested.

iFrames => decoded independently
other frames => rely on nearby frames

By default, we use the nearest iframe.  You can set `requestedTimeTolerance` to customize.  Consider setting wide tolerances that still give you the result you're looking for.  
```swift
func thumbnail() async throws -> UIImage {
    let generator = AVAssetImageGenerator(asset: asset)
    generator.requestedTimeToleranceBefore = .zero
    generator.requestedTimeToleranceAfter = CMTime(seconds: 3, preferredTimescale: 600)
    let thumbnail = try await generator.image(at: time).image
    return UIImage(cgImage: thumbnail)
}
```

To get a series of thumbnails, new API

```swift
func timelineThumbnails(for times: [CMTime]) async {
    for await result in generator.images(for: times) {
        switch result {
        case .success(requestedTime: let requestedTime, image: let image, actualTime: _):
            updateThumbnail(for: requestedTime, with: image)
        case .failed(requestedTime: let requestedTime, error: _):
            updateThumbnail(for: requestedTime, with: placeholder)
        }
    }
}
```

supercedes old aPI which had gotchas.

Lets you await the next element after iteration. 

```swift
func timelineThumbnails(for times: [CMTime]) async {
    for await result in generator.images(for: times) {
        updateThumbnail(for: result.requestedTime, with: (try? result.image) ?? placeholder)
    }
}
```

Directly access the properties.  Can throw errors if generation fails.

[[Meet AsyncSequence]]

There are some other synchronous areas of AVFoundation.

## AVMutableComposition
`.insertTimeRange` needs information about asset's tracks.  It synchronously inspects the track, so if the tracks aren't loaded, they will be sync loaded.

Previously, the solution would have been to await loading the tracks prior.  However, this year we're introducing an async version of `insertTimeRange` which loads tracks async as needed.
```swift
let composition = AVMutableComposition()
try await composition.insertTimeRange(timeRange, of: asset, at: startTime)
```

## AVVideoComposition
```swift
let videoComposition = try await AVVideoComposition .videoComposition(withPropertiesOf: asset)

try await videoComposition.isValid(for: asset, timeRange: range, validationDelegate: delegate)
```

New this year, properties of asset ocnstructor and `isValid(for:)` has async.  Don't need to preload these either.

When you need to load properties yourself, let's get a refresher of


# Asset inspection

two ways to inspect.

async key value loading
last year, we introduced async loading.

```swift
asset.loadValuesAsynchronously(forKeys: ["duration", "tracks"]) {
    guard asset.statusOfValue(forKey: "duration", error: &error) == .loaded else { ... }
    guard asset.statusOfValue(forKey: "tracks", error: &error) == .loaded else { ... }
    myFunction(thatUses: asset.duration, and: asset.tracks)
}

let (duration, tracks) = try await asset.load(.duration, .tracks)
myFunction(thatUses: duration, and: tracks)
```

typos in string keys are tough to catch.  Easy to forget to add new properties, forget async loading , etc.  For these reasons, we're deprecating key value loading, in favor for async load.

* typesafe identifiers
* directly return loaded values
* checked at compile time
The **only** recommended way to async inspect avasset, track, etc.  A handful of these properties will still offer sync.  Some data is already available in memory.

## ex: AVMutableComposition
```swift
// videoTrack1: AVAssetTrack, videoTrack2: AVAssetTrack

// Create a composition and add an empty track
let composition = AVMutableComposition()
guard let compositionTrack = composition
    .addMutableTrack(withMediaType: .video,
                     preferredTrackID: 1) else { return }

// Append the first 5 seconds of track 1
try compositionTrack
    .insertTimeRange(firstFiveSeconds,
                     of: videoTrack1, at: .zero)

// Append the first 5 seconds of track 2
try compositionTrack
    .insertTimeRange(firstFiveSeconds,
                     of: videoTrack2, at: fiveSeconds)
myFunction(thatUses: composition.duration,
           and: composition.tracks)
```

Not loading any data.  WE add a new track segment to point to the desired track.  Then we append part of the second track in the same way.  Since composition itself is backed by an in-memory structure, we can inspect synchronously.

Sync will remain available on some classes.

Concurrency resources
[[Swift concurrency update a sample app]]
[[What's new in AVFoundation]]
# Custom data loading
When you create an AVAsset, media can be on network, or local.  If on network, AVAsset will dynamically cache a certain amount of data.
Local => AVAsset can bypass the cache and load data as needed.  You might not be able to give AVAsset a direct pointer.  ex, custom file.
AVASsetREsoruceLoader => request arbitrary bytes.  Since the asset is no longer hadnling reading data, it can't predict how long it will take to load.  It assumes the network case, and we cache data.

This year, if your media is locally available, you can enable `entireLengthAvailableOnDemand` for local resources.  

## Local resource loading
* reduce memory usage
* improve playback startup time
* only for local media

# Wrap up
Use async/await to keep your app responsive
Increase tolerances for faster results
 
















* https://developer.apple.com/forums/tags/wwdc2022-110379
* https://developer.apple.com/forums/create/question?&tag1=307&tag2=552030
* https://developer.apple.com/documentation/avfoundation/media_reading_and_writing/creating_images_from_a_video_asset
* https://developer.apple.com/documentation/avfoundation/media_assets/loading_media_data_asynchronously
