# Overview
Waiting is not fun.  Any time you ask users to wait, you're increasing frustration.

ex, looking for a perfect app.  Tap Get.  

> with every moment, your level of excitement increases


You may find yourself having to wait as the app downloads.   Then you launch the app and there's more downloading.  Couldn't it have downloaded this after the app was installed?

# Background assets
* new framework
* Flexible with existing workflows
* Out of band content
* Execution outside of an app's lifecycle
* ease of use

## app extension
extension will execute
* after app isntallation by the app store or testflight
* after an app update
* Periodically in the background
* Short and limited runtime
	* System may terminate the extension
* Extension runtime is based on app usage
	* less frequent if your app isn't used much.

# Adopting the framework
Download manager
* singleton for managing assets
* Using the manager, you can schedule the download of your assets in either foregroundo f bg
* retrieve current download operations
* cancel downloads
* Synchronize between app and extension

```swift
// Getting started with Background Assets
import BackgroundAssets

let url = URL(string: "https://cdn.example.com/large-asset.bin")!
let appGroupIdentifier = "group.WWDC.AssetContainer"
let download = BAURLDownload ( identifier: "Large-Asset",
                               request: URLRequest(url:url),
                               applicationGroupIdentifier:
                                appGroupIdentifier )

let manager = BADownloadManager.shared
manager.delegate = self  // BADownloadManagerDelegate protocol

// Schedule download at an opportunistic time determined by the system
do {
    try manager.schedule(download)
} catch {
    print("Failed to schedule download. \(error)")
}

// or Schedule download in foreground
do {
    try manager.startForegroundDownload(download)
} catch {
    print("Failed to start foreground download. \(error)")
}

// or Promote downloads to foreground.
do {
    for download in try await manager.fetchCurrentDownloads) {
       try manager.startForegroundDownload(download)
    }
} catch {
    print("Failed to promote downloads to foreground \(error)")
}
```

Having your app and extension in the same group lets you both manage assets.  aDd one from signing and capabilities section.  allows 2 or more apps ot acces the same resources, or in this case app + extension.

In this example, we'll be focusing on the most common one: BAURLDownload.  Takes a URL and applicationGroupIdentifier.  Use this identifier to track the download across multiple launches.

Engine will nto allow more than one download with the same identifier.

Receives messages about donwloads that have been scheduled.  If for any reaosn a download cannot bescheduled, an error is thrown.
foreground enables your download to begin immediately.  Similar to Default session configuration.  Your app can promote any downloads from bg to fg.

Performing a foreground download is not available from the extension, it can only be initiated from the app.  Since extensions never present UI, they can only schedule in the bg.  If your app wants to promte to ft, see sample above.

If the download was backgroudned, it will be paused and then resumed in the foreground.  

Delegate receives callbacks for all schedule dassets.  If there are numerous downloads that were scheduled, callbacks will be received for all of them.  Use unique identifier.
* callbacks are not enqueued, called immediately
	* If your app does not handle a delegate method, then your extension will wait to process the mesasge.
	* Expect yuor extension to be sent messages if you don't have a delegate in the app
* App will get messages if it's running, not the extension
* System launches extension if app does not handle messages.
* Extension only launches for BADownloaderExtension entrypoints.
* Extensions can use delegates with BADownloadManagter
	* app and extension will receive duplicate messages


```swift
// BADownloadManager protocol definition
public protocol BADownloadManagerDelegate : NSObjectProtocol {
    optional func downloadDidBegin(_ download: BADownload)

    optional func downloadDidPause(_ download: BADownload)

    optional func download(_ download: BADownload,
                           bytesWritten: Int64,
                           totalBytesWritten: Int64,
                           totalExpectedBytes: Int64)

    optional func download(_ download: BADownload, didReceive challenge: URLAuthenticationChallenge) async -> (URLSession.AuthChallengeDisposition, URLCredential?)

    optional func download(_ download: BADownload, failedWithError error: Error)

    optional func download(_ download: BADownload, finishedWithFileURL fileURL: URL)
}
```

Please do not duplicate the file unless you delete the originating file.


Remember, this protocol is for receiving messages, but it is not the entrypoint for the extension.

# Extension overview
Schedule downloads before the user has launched your app.  Ensure that your assets are in place.

* New app extension
* Runs during app install and update
* Periodically to check for new assets
* Short lived and sandboxed

**App** info plist must contain (reviewed by app review)
* BAInitialDownloadRestrictions
	* BADownloadAllowance => max download size you're requesting to make during initial app install.  Sum of all files that you are requesting to download, not the size of each file.
	* BADownloadDomainAllowList => Prefix wildcards.  List of host names you may download from.  Keys are only enforced after first app install.  **When app is launched, these are no longer enforced.**
* BAMaxInstallSize => Additional storage for these assets.  Final, extracted, uncompressed size.  

* brokered by the system, not your app
* For quick work to schedule downloads
	* keep work done to a minimum
	* don't kick off decompression, etc.
* Access to download manager
* Use app groups to share files

```swift
// BADownloaderExtension protocol definition
public protocol BADownloaderExtension : NSObjectProtocol {
    optional func applicationDidInstall(metadata: BAApplicationExtensionInfo)

    optional func applicationDidUpdate(metadata: BAApplicationExtensionInfo)

    optional func checkForUpdates(metadata: BAApplicationExtensionInfo)

    optional func download(_ download: BADownload, didReceive challenge: URLAuthenticationChallenge) async -> (URLSession.AuthChallengeDisposition, URLCredential?)

    optional func backgroundDownloadDidFail(failedDownload: BADownload)

    optional func backgroundDownloadDidFinish(finishedDownload: BADownload, fileURL: URL)
  
    optional func extensionWillTerminate()
}
```

These fucntions can be invoked even if your **app** scheduled the download.  extension might be expected to service the download.

What if they're both running simultaneously?

1.  System wakes extension to check for updates
2. Extension is going to use BADownloadManager
3. Extensionw ill download a catalog file that will have more assets
4. Entrypoints are not guaranteed to be invoked immediately
5. Extension finds out download is finished.  Can read file, schedule more downloads, etc.
6. Read and delete the file. 


What happen sif your app launches during all this?
1.  App wants to know if it has updated content.  Since app was launched before catalog is finished, it will fetch active downloads and see something inflight.
2. But both extension and app will receive finish message.  We have a data race ont he file being downloaded.
3. Both app and extensiont ry to read/delete the file.
4. App or extension might find the file missing.

we provide a way to synchronize between app and extension.


```swift
// Synchronizing between app and extension
func download(_ download: BADownload, finishedWithFileURL fileURL: URL) {
    let manager = BADownloadManager.shared
    manager.withExclusiveControl { error in
        guard error == nil else {
            print("Unable to acquire exclusive control \(String(describing: error))")
            return
        }
        // Exclusive control acquired
        // All code in this scope ensures mutual exclusion between extension and app
        do {
            let data = try Data(contentsOf: fileURL, options: .mappedIfSafe)
            // Do something with memory mapped data
            try FileManager.default.removeItem(at: fileURL)
        } catch {
            print("Unable to read/cleanup file data. \(error)")
        }
    }
}
```

Ensuring mutual exclusion of the file.  `withExclusiveControl`.  Code in scope is guaranteed to be mutually exlcusive with other code that requires control.  If extension acquires exlcusive control first, the app wil wait until the extension is terminated.

> Acquiring exclusive control can fail.  It is extremely unlikely that this will occur, but you should handle it.


Read ocntents of file and clean it up if you hcoose.  Other side should check if the file exists, or read results somewhere.  Figure it out, I dunno


# Best practices
* Use BADownloadManager from app and extension
* Extension runs when app is in background
* App should foreground downloads
* Use the exclusive control APIs for synchronization

# Wrap up
* minimize waiting
* Adopt background assets
* Check out the documentation
* Provide us feedback

[[Accelerate networking with HTTP3 and QUIC]]
[[Introducing on demand resources - 15]]











* https://developer.apple.com/documentation/foundation/nsbundleresourcerequest
* https://developer.apple.com/documentation/backgroundassets
