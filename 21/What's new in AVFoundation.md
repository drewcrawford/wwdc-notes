# AVAsset async inspection
* Local movie file
* Remote movie file
* HLS

2 important things to keep in mind.
* On demand
	* Movies are large
* Asynchronous
	* Network I/O takes time

```swift
func inspectAsset() async throws {
	let asset = AVAsset(url: movieURL)
	let duration = try await asset.load(.duration)
	myFunction(thatUses: duration)
}
```

[[Meet asyncawait in Swift]]

For now, a quick way of understanding property loading, think of the await keyword as two parts.  First, the part before async, then the part after.

Can load values of multiple properties by passing in more than one property identifier.  Here we load both duration and tracks at the same time.  

Can also check status without waiting

```swift
switch asset.status(of: .duration) {
case .notYetLoaded:
	// This is the initial state after creating an asset. 
case .loading:
	// This means the asset is actively doing work.
case .loaded(let duration):
	// Use the asset's property value.
case .failed(let error):
	// Handle the error.
}
```

Each property starts as `notYetLoaded`.  Inspection happens on demand, so until you ask to load a property value the asset won't have done other work.

Quite a few properties that can be loaded async. 

old API
```swift
asset.loadValuesAsynchronously(forKeys: ["duration", "tracks"]) {
	var error: NSError?
	guard asset.statusOfValue(forKey: "duration", error: &error) == .loaded else { ... }
	guard asset.statusOfValue(forKey: "tracks", error: &error) == .loaded else { ... }
	let duration = asset.duration
	let audioTracks = asset.tracks(withMediaType: .audio)
	// Use duration and audioTracks.
}
```

Beware of blocking I/O.  New APIs eliminate the misuse, so the old API is deprecated.  We recommend you migrate.

Now `status(of:)` enum contains bundled value.

`load(_:)` again will return the cached value
One caveat, because the `load` method is async, it can only be called from an async context.  So if you need the value from synchronous property, can get status and assert is `.loaded()`.

Possible for a property to become failed even after it's been loaded.

No replacement for synchronous property or blocking method.

# Video compositing with metadata
Process of taking multiple video tracks and composing into a single stream.

## Video composition with metadata
* Per-frame metadata
* Available in frame composition callback
* Timed metadata track
	* `AVAssetWriterInputMetadataAdaptor`

`videoComposition.sourceSampleDataTrackIDs` tell us about metadata tracks
Each video composition instructions should set `requiredSoruceSampleTrackIDs`
Do both setup steps *or you won't get any metadata*!

```swift
// This is an implementation of a AVVideoCompositing method:
func startRequest(_ request: AVAsynchronousVideoCompositionRequest) {
	for trackID in request.sourceSampleDataTrackIDs {
		let metadata: AVTimedMetadataGroup? = request.sourceTimedMetadata(byTrackID: trackID)
		// To get CMSampleBuffers instead, use sourceSampleBuffer(byTrackID:).

	}

	// Compose input video frames, using metadata, here.

	request.finish(withComposedVideoFrame: composedFrame)
}
```

# Caption file authoring
2 file formats.
* Subtitles (iTuens Timed Text) itt
* Closed captions
	* Scenarist Closed Caption (.scc)
* Author
* Ingest
* Preview

* AVCaption
	* Text
	* Position
	* Styling
* AVAssetWriterInputCaptionAdaptor
* AVCaptionConversionValidator
	* Formats have various requirements like they have a max bitrate
	
## Ingest
* AVAssetReaderOutputCaptionAdaptor
* AVCaptionRenderer
	* CGContext

# Wrap up
* Inspecting assets
* On demand and asynchronously
* New asynchronous API
* Migrate from old, synchronous API
* Custom video compositing with metadata
* Caption file authoring

https://developer.apple.com/documentation/avfoundation/media_assets_and_metadata/loading_media_data_asynchronously
https://developer.apple.com/documentation/avfoundation

