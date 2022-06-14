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
