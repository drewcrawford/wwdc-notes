#macOS 

# Introduction
* FileProvider integrates Cloud Storage into APFS
* On-demand downloads of user files and folders
* User space replacement for FUSE or KAUTH
	* kexts are deprecated
* You focus on providing data and maintaining sync
* System handles file system integration

* You implement `NSFileProviderReplicatedExtension`
* You create a domain
	* System assigns location in finder sidebar
	* System assigns a root directory on disk
* User can start interacting with domain immediately

## Dataless files
The root directory is initially dataless
* New feature in APFS
* Identified by SF_DATALESS, see chflags(2)
* POSIX reads (and file coordinations) trigger materialization
* By triggering callbacks in `NSFileProviderReplicatedExtension`
# User flows
* Downloading a file
* Listing a directory
* Propagating a remote change
* Propagating a local change

## Downloading a file
1.  read
2. ` NSFileProviderReplicatedExtension.fetchContents(...)`
3. You do whatever it is you do
4. `completionHandler`
5. System replies to read.
6. Subsequent reads do not involve your extension

## Listing a directory
Very similar
Paginated.  Return less than the full set of items and the system will pick up enumerating.
Once all pages have been enumerated, will allow the original call to go through.

Subsequent readdir calls will use contents from disk.

What if contents change?

You have to inform the system.
## Propagating remote changes
1.  Push notification
2.  `NSFileProviderManager signalEnumerator(for: .workingSet)`
3.  System enumerates the items with `syncAnchor`
4.  Yu return changed items
5.  Give new `syncAnchor`
6.  Coordinated update


## Propagating local changes
1.  Write
2.  `NSFileProviderReplicatedExtension.modifyItem`
3.  Note that changes are aggregated e.g. package files etc.
4.  You update
5.  System will hand you a clone that will be consistent
6.  completionHandler
7.  Conflicts.  Since you pass the final state back, system will update local state to match the cloud.

## Eviction
The system may evict local files to reclaim disk space.  You are not told.  LRU.


## Review
* Eviction => local file to dataless
* Download => dataless file to local

System will only evict a file that you report as uploaded.  So there are 2 sorts of local files, uploaded or not.

There are methods to trigger eviction.
# Demo

# Order of implementation
* Show up in Finder
	* `NSFileProviderManager.add()`
* Sync down a directory listing
	* `class Item: NSFileProviderItemProtocol`
	* `NSFileProviderEnumerator`
* Download a file
	* `fetchContents(for:)`
* Sync down a remote change
	* `currentSyncAnchor`
	* `enumerateChanges()`
* Sync up a local change
	* `createItem` 
	* `modifyItem` 
	* `delete`
# System integration
Icon decorations can be used to visually decorate items.
* Badge
* folder badge
* sharing

Info.plist

## Contextual menu actions
FileProvider
FileProviderUI

Info.plist

## Pre-flight alerts
* Warn user before they take an action with unintended cosnequences

Configured in Info.plist

# Next steps
* Download the FruitBasket sample code
* Use the Xcode Fileprovider target template
* Implement the method stubs

https://developer.apple.com/documentation/fileprovider/macos_support
https://developer.apple.com/documentation/fileprovider/macos_support/syncing_files_on_macos
https://developer.apple.com/documentation/fileprovider

