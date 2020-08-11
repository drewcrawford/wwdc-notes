#hls 

* Introduced in 2016
* Reuse existing media library
* Supports FairPlay streaming protection
* Downloads in the background

# Should I use offline HLS?
Yes.


1. Create a download task
2. Monitor the download task
3. Play the downloaded content

# Download task
Inherits features of URLSession.
* scheduled to optimize system resources.  In certain cases, the download may not start immediately.
* Retried for transient network errors

# `AVAssetDownloadTask`
various playlists, etc.

Download 1 combination of video, audio, subtitle renditions.
```swift
let hlsAsset = AVURLAsset(url: assetURL)

let backgroundConfiguration = URLSessionConfiguration.background(
    withIdentifier: "assetDownloadConfigurationIdentifier")
let assetURLSession = AVAssetDownloadURLSession(configuration: backgroundConfiguration,
    assetDownloadDelegate: self, delegateQueue: OperationQueue.main())

// Download a Movie at 2 mbps
let assetDownloadTask = assetURLSession.makeAssetDownloadTask(asset: hlsAsset,
    assetTitle: "My Movie", assetArtworkData: myAssetArtwork,
    options: [AVAssetDownloadTaskMinimumRequiredMediaBitrateKey: 2000000])!
assetDownloadTask.resume()



// AVAssetDownloadTask uses automatic media selection
```
What does audiomatic media selection?
Device region and localization preferences.  So if device region is france, we will try to get french audio, etc.

Monitor progress

```swift
// Monitor AVAssetDownloadTask
public protocol AVAssetDownloadDelegate: URLSessionTaskDelegate {


	// Use to monitor progress
	func urlSession(_ session: URLSession, assetDownloadTask: AVAssetDownloadTask,
		didLoad timeRange: CMTimeRange, totalTimeRangesLoaded loadedTimeRanges: [NSValue],
		timeRangeExpectedToLoad: CMTimeRange)


	// Listen for completion
	func urlSession(_ session: URLSession, task: URLSessionTask,
		didCompleteWithError error: Error?)

}
```

```swift
// Monitoring
MyAssetDownloadDelegate: NSObject, AVAssetDownloadDelegate {
    func urlSession(_ session: URLSession, assetDownloadTask: AVAssetDownloadTask,
didLoad timeRange: CMTimeRange, totalTimeRangesLoaded loadedTimeRanges: [NSValue], timeRangeExpectedToLoad: CMTimeRange) {

		// Convert loadedTimeRanges to CMTimeRanges
		var percentComplete = 0.0
		for value in loadedTimeRanges {
			let loadedTimeRange: CMTimeRange = value.timeRangeValue 
			percentComplete += CMTimeGetSeconds(loadedTimeRange.duration) /
				CMTimeGetSeconds(timeRangeExpectedToLoad.duration)
		}
		percentComplete *= 100
		print("percent complete: \(percentComplete)")
	}
}
```
What if I want to download multiple renditions?

# `AVAggregateAssetDownloadTask`
Can specify multiple variants.

```swift
let hlsAsset = AVURLAsset(url: assetURL)
let myMediaSelections = [] // audio media-selections followed by subtitle media-selections

guard hlsAsset.statusOfValue(forKey: "availableMediaCharacteristicsWithMediaSelectionOptions", error: nil) 
   == AVKeyValueStatus.loaded else { return }

let mediaCharacteristic = //AVMediaCharacteristic.audible or AVMediaCharacteristic.legible
let mediaSelectionGroup = hlsAsset.mediaSelectionGroup(forMediaCharacteristic: mediaCharacteristic)
if let options = mediaSelectionGroup?.options {
    for option in options {
        // chose your media selection option
        if /* this is my option */ {
            let mutableMediaSelection = hlsAsset.preferredMediaSelection.mutableCopy()
            mutableMediaSelection.select(option, in: mediaSelectionGroup)
            myMediaSelections.append(mutableMediaSelection)
        }
    }
}
```

```swift
let hlsAsset = AVURLAsset(url: assetURL)
let myMediaSelections = ...

let backgroundConfiguration = URLSessionConfiguration.background(
    withIdentifier: "assetDownloadConfigurationIdentifier")
let assetURLSession = AVAssetDownloadURLSession(configuration: backgroundConfiguration,
    assetDownloadDelegate: self, delegateQueue: OperationQueue.main())

// Download a Movie at 2 mbps
let aggDownloadTask = assetURLSession.aggregateAssetDownloadTask(with: hlsAsset,
    mediaSelections: myMediaSelections,
    assetTitle: "My Movie",
    assetArtworkData: myAssetArtwork,
    options:[AVAssetDownloadTaskMinimumRequiredMediaBitrateKey: 2000000])!
aggDownloadTask.resume()
```


Do monitor progress, split by type.

* first media selection is typically video, and will take a longer time.  70% in this example
* audio - 20%
* subtitles - 10%


```swift
// Monitor AVAggregateAssetDownloadTask
public protocol AVAssetDownloadDelegate: URLSessionTaskDelegate {

	// Use to monitor progress
	func urlSession(_ session: URLSession, 
		aggregateAssetDownloadTask: AVAggregateAssetDownloadTask, 
		didLoad timeRange: CMTimeRange, totalTimeRangesLoaded loadedTimeRanges: [NSValue], 
		timeRangeExpectedToLoad: CMTimeRange, 
		for mediaSelection: AVMediaSelection
	)

	// Listen for completion for each media selection
	func urlSession(_ session: URLSession, 
		aggregateAssetDownloadTask: AVAggregateAssetDownloadTask, 
		didCompleteFor mediaSelection: AVMediaSelection)

    // In case of audio rendition, expect calls once for stereo followed by once for multichannel rep.
}
```

Your app may get backgrounded during download.  Download still occurs.  Need to restore tasks on app launch.
```swift
// Restore Tasks on App Launch
class MyAppDelegate: UIResponder, UIApplicationDelegate {
	func application(_ application: UIApplication, 
			didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
		let configuration = URLSessionConfiguration.background(withIdentifier:
			"assetDownloadConfigurationIdentifier")
		let session = URLSession(configuration: configuration) 
		session.getAllTasks { tasks in
			for task in tasks {
				if let assetDownloadTask = task as? AVAssetDownloadTask {
					// restore progress indicators, state, etc...
				} 
			}
		}
	}
}
```

# Get started with Offline HLS
Where is the download on disk?  In case of `AVAssetDownloadTask`, we get location once download is complete.
In the aggregate case, we get out of `willDownloadTo` delegate callback.


```swift
// Store the download location
public protocol AVAssetDownloadDelegate: URLSessionTaskDelegate {

	// AVAssetDownloadTask
	func urlSession(_ session: URLSession, 
		assetDownloadTask: AVAssetDownloadTask, 
		didFinishDownloadingTo location: URL)

	// AVAggregateAssetDownloadTask
	func urlSession(_ session: URLSession, 
		aggregateAssetDownloadTask: AVAggregateAssetDownloadTask, 
		willDownloadTo location: URL)

}
```

May want to store this location.  

```swift
// Instantiating Your AVAsset for Playback

// 1) Create Asset for AVAssetDownloadTask
let networkURL = URL(string: "http://example.com/master.m3u8")!
let asset = AVURLAsset(url: networkURL)
let task = assetDownloadSession.makeAssetDownloadTask(asset: asset, assetTitle: "My Movie",
assetArtworkData: nil, options: nil)


// 2) Re-use Asset for Playback, Even After Task Restoration at App Launch
let playerItem = AVPlayerItem(asset: task.urlAsset)


// Reusing asset, will allow AVFoundation to optimize resources between playback and download in cases where the playback happens before the download is complete.
```

This (reusing asset) optimizes the case where we are playing while download is in progress.

Alternatively, you may be playing long after download is complete.  Here you may not have `AVAsset` or `AV*AssetDownloadTask` lying around.


```swift
// Create using file URL

let fileURL = URL(fileURLWithPath: self.savedAssetDownloadLocation)
let asset = AVURLAsset(url: fileURL)

let playerItem = AVPlayerItem(asset: task.urlAsset)
```

How to find out what can be played offline?  Ask `AVAssetCache`

```swift
// What can I play offline?

public class AVURLAsset {

	public var assetCache: AVAssetCache? { get }

}

public class AVAssetCache {

	public var isPlayableOffline: Bool { get }

	public func mediaSelectionOptions(in mediaSelectionGroup: AVMediaSelectionGroup)
		-> [AVMediaSelectionOption]

}
```

# Protect using Fairplay Streaming

* introduced in 2016
* Protect your Offline HLS media content

[[AVContentKeySession Best Practices - 18]]

## create offline key
* Fairplay Streaming lets your key server set expiration date for offline keys
* Does not expire in between a playback session
* Remember to create a new offline key before the expiration
* May want to invalidate keys when content is deleted for example

```swift
// Invalidate Offline Key

public class AVContentKeySession {

	func invalidatePersistableContentKey(_ persistableContentKeyData: Data, 
		options: [AVContentKeySessionServerPlaybackContextOption : Any]? = nil, 
		completionHandler handler: @escaping (Data?, Error?) -> Void)


	func invalidateAllPersistableContentKeys(forApp appIdentifier: Data, 
		options: [AVContentKeySessionServerPlaybackContextOption : Any]? = nil, 
		completionHandler handler: @escaping (Data?, Error?) -> Void)


}
```

## movie rentals
* you want to allow your users to download rental movies
* two expiration windows, playback and rental
* [[AVContentKeySession Best Practices - 18]]

## Custom protocol for URL resources
ex
* `myscheme://my-playlist-1`
* `skd://key-2`

Starting from iOS 14, you have 30 secs to do key requests in the background from these custom protocols

# best practices
## optimize download time
Option for fast download
Option for best quality download (takes more time)

```swift
// Quality Selection

public class AVAssetDownloadTask {

	public let AVAssetDownloadTaskMinimumRequiredMediaBitrateKey: String

	//Starting in iOS 14

	public let AVAssetDownloadTaskMinimumRequiredPresentationSizeKey: String

	public let AVAssetDownloadTaskPrefersHDRKey: String

}
```

```swift
// Multichannel Audio Selection

public class AVAssetDownloadTask {

	public let AVAssetDownloadTaskPrefersMultichannelKey: String

}
```
Why download both?  We believe taht the stereo better reflects artistic intent.  e.g., dialog promenance.  You may want to expierment and choose appropriately.

I did not follow this answer at all.
## storage management
The OS can delete offline assets while the app is not running
For example, when storage is running low on device. ex, during software updates.
Allows users to delete media content through settings.  Asset image and title will be displayed here.

```swift
// AVAssetDownloadStorageManager 
// Get the singleton 
let storageManager = AVAssetDownloadStorageManager.shared()
 
// Create the policy 
let newPolicy = AVMutableAssetDownloadStorageManagementPolicy() 

newPolicy.expirationDate = myExpiryDate

newPolicy.priority = .important 

// Set the policy
storageManager.setStorageManagementPolicy(newPolicy, forURL: myDownloadStorageURL)
```

assets will be purged based on expiration date, and then based on priority

keep downloaded assets at the system provided location
be prepared for the system to delete your assets to reclaim disk space.

# wrap up
* 2 variants of download tasks
	* `AVASSEtDownloadTask` automatic media selection
	* `AVAggregateAssetDownloadTask` you specify variants
* FairPlay streaming protection

# resources
https://developer.apple.com/documentation/avfoundation/media_playback_and_selection/using_avfoundation_to_play_and_persist_http_live_streams
