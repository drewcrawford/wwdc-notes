Discover how you can sync files faster and more efficiently within your iPhone and iPad apps when you create a File Provider extension. Sync up with the File Provider team and learn how to build a modern File Provider for iOS. We'll show you how to architect your app to support seamless file sync, uploads, and downloads. And we'll explore how you can go stateless and fortify your file provider against unexpected conditions. To get the most out of this session, we recommend having experience with File Providers on macOS.

# Desktop class sync
Big Sur introduced declarative api for syncing files to your mac.  

Shared location on the filesystem
Accessing all types of file system objects
Background sync for consistency


Your app extensionf ulfills requests
enumerate items
fetch items
update lists of items

System maintains on-disk structure and consistency
Users will retry a download if necessary
Upload operations are retried if they fail
Local version is maintained during upload
Protected by atomic file system operations and coordination

# Modern file providers

Storage management
Transparently evict files that don't have local changes based on disk usage and LRU status.
files that are fully offloaded do not count against your app.

# Architecture of your app
Strict separation of concerns.

System manages structures on disk.
Your extension should perform tasks to sync up/down.
System tracks all state of hierarchy and which parts require sync.
Extension can be very lightweight.   Doesn't have to track any item-specific state at all.
Application is not responsible for any syncing.  ideally it doesn't have to talk to server at all.  Instead, it interacts with extension through 2 mechanisms

1.  Indirectly, same way that any other app does.  `NSfileProviderManager.userVisibleLocation(for:)`.  Then accessible with regular FS API.
2. `NSFileProviderManager.service(named:)` to get direct XPC.  Sharing files or resolving conflicts.

Can also be used by FPUI extensions.  




### Implement a Progress Cancellation Handler - 6:04
```swift
// Implementing a progress cancellation handler
public func modifyItem(_ item: ..., completionHandler: (..., Error?) -> Void) -> Progress {
    let progress = Progress()
    let uploadTask = Task {
        do {
            // ...
            try Task.checkCancellation()
            // ...
        } catch let error {
            completionHandler(nil, [], false, error)
        }
    }
    progress.cancellationHandler = {
        uploadTask.cancel()
    }
    return progress
}
```


# Stateless providers

Upload management

Extension must report progress!
If an upload doesn't progress, it will be cancelled.  System provides a grace period to wind down upload cleanly, but if it takes too long you will be terminated.

In case of uploads, use modifyItems.  Cancel the actual upload work you were performing.  Of course, you also need to call completion handler, to signal that cancellation error occurred.  Here we use an async task cancellation. Can also call manually.

Push.
Your app will not be running to check for changes.
Support push notifications.  PushKit exposes a specific push type for file provider.

Note that we throttle pushes in some cases.  

### Register for Push Notifications - 6:53
```swift
// Registering for push notifications
import PushKit

let pushRegistry = PKPushRegistry(queue: queue)
pushRegistry.delegate = self
pushRegistry.desiredPushTypes = Set([PKPushType.fileProvider])

...

// On the server: push
//
// {
//    "container-identifier" =    "NSFileProviderWorkingSetContainerItemIdentifier"
//    "domain" =    "<domain identifier>"
// }
//
// with topic "<your application identifier>.pushkit.fileprovider"
```


# Demo

App can operate on all items in the folder.  Drag folder from files app to batch editor.

Say I want to implement dragging an item.


### Drag and Drop: Implement Dragging - 8:53
```swift
// Sending out drags
var body: some View {
    Text("🥐")
        .onDrag {
            let itemProvider = NSItemProvider()
            itemProvider.registerFileRepresentation(for: .folder, openInPlace: true) { completionHandler in
                self.manager.getUserVisibleURL(for: folderItemID) { fileURL, error in
                    guard let fileURL = fileURL else {
                        completionHandler(nil, false, error)
                        return
                    }
                    completionHandler(fileURL, true, nil)
                }
                return Progress()
            }
            return itemProvider
        }
}
```


### Drag and Drop: Implement Dropping - 9:24
```swift
// Receiving drops
var body: some View {
    Text("🥬")
        .onDrop(of: [.folder], isTargeted: $dropTarget) { providers in
            guard let prov = providers.first(where: { provider in
                !provider.registeredContentTypes(conformingTo: .folder).isEmpty
            }) else {
                return false
            }
            prov.loadFileRepresentation(for: .folder, openInPlace: true) { url, inPlace, err in
                guard let url = url else {
                    return
                }
                Task {
                    url.startAccessingSecurityScopedResource()
                    // use URL
                    url.stopAccessingSecurityScopedResource()
                }
            }
            return true
        }
}
```

# Wrap up
* download the sample code
* Updated xcode template

[[Sync files to the cloud with FileProvider on macOS]]

# Resources

* https://developer.apple.com/documentation/fileprovider
* https://developer.apple.com/documentation/FileProviderUI
* https://developer.apple.com/documentation/UserNotifications/sending-notification-requests-to-apns
* https://developer.apple.com/documentation/fileprovider/replicated_file_provider_extension/synchronizing_files_using_file_provider_extensions
