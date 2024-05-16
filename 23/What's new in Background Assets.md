Waiting is no fun! Discover how Background Assets can help your app download content before it even launches. We'll show you how to integrate Background Assets into an existing app, explore when to use essential or non-essential assets, and learn how to make debugging your extension a breeze.

info plist requirements (old)

[[Meet Background Assets]]

# Feature recap

* helps to prevent waiting after app launch
* Framework with a paired extension
* iOS 16.1.  Alongside macOS ventura.
* Out-of-band content
* Execution outside of app's lifecycle
* Supported on macOS, iOS, and iPadOS

fetches content before launch:
* during app install by appstore/testflight
* Periodically requests new content to download
* Services downloads while the app is not running
* Provides limited runtime to manage content
* placed into a restricted sandbox.  If you find an api isn't available, tell us with FA.

Lifecycle.

1.  app store installs
2. background assets system service is notified to prevent app from launching
3. system inspects app bundle, info plist
4. system downloads manifest and reports progress back to app store
5. once manifest has been downloaded, system wakes extension with content request.
6. extension uses the manifest to determine urls, filesizes, assets to schedule for download.  Then we return downloads as a set of `BADownloads`
7. system pauses/terminates extension
8. downloads begin
9. you're notified when complete.

Period check, nearly identical.  Device determines when the event will occur.  

Runtime limitations
* exist to preserve power and performance
* don't exceed a few MB of memory
* Memory map large files you need to read, as memory-mapped data does not count against the limitation.
* Few minutes of runtime per day initially
* changes based on app usage.  System throttles launches if you're not used.
* Commonly used apps may get additional runtime
* Function scope in `BADownloaderExtension`.
* Runtime during `withExclusiveControl()`.  Up until completion handler is invoked/returned.

Controlled by user:
* low power mode
* background app refresh turned off

`downloads(for request:manifestURL:extensionInfo:)`.

manifest is commonly used to compare what is currently downloaded against what's available on server.

but suppose that downloadsForManifest takes forever.  This exceeds extension's runtime and extension will be terminated.  Important to remember that runtime is calculated fro the moment it's called up until it exits scope and returns.

instance variables or in-memory state are not preserved when terminated.

Singleton for managing assets: BADownloadManager.
Schedule downloads
Manage in-flight downloads.
Acquire exclusive control
Receives messages over the extension

* downloaded assets are marked purgeable
* Modified assets lose their purgeability.  Think carefully about modifying assets.
* Increase the size of backups
* may prevent security updates
* Place assets into `.cachesDirectory`.  

# What's new

## essential downloads

* Downloads during app install or update
* Integrates download progress into: iOS home screen, launchpad, app store
* Blocks app launch until download complete

server should support HTTP range. 
Essential downloads take priority during install.

As a side note, important that your extension vend downloads quickly.  

extension may receive multiple authentication challenges.

failed downloads can be re-enqueued as non-essential.
As your extension is receiving completion for essential, you will start getting non-essential.

1.  app download
2. app install
3. essential assets

BAEssentialDownloadAllowance is used to setup the progress indicator.  Then we use filesize to determine how much is actually being downloaded.  Get your download allowance close to what's being downloaded.

Everything we've discussed can be disabled!  Can disable 'in-app content'.  While it doesn't  disable everything, it does prevent essential assets from downloading, and the extension running before launch.

essential *but not a requirement for your app to launch*.  Your app must handle flows where essential assets are not on the device yet.

we introduced this in 16.4.

New initializer on BAURLDownload to create downloads as essential.  Download will fail if you don't provide the right size!  If you dn't know the size, use BAManifest.

`removingEssential` to re-queue the task, etc.

Required keys.



|Key|Type|Description|
|-|-|-|
|BAInitialDownloadRestrictions|Dictionary|The restrictions that apply to the set of assets that download prior to first app launch.
|BADownloadAllowance|Number|The combined size of the initial set of non-Essential asset downloads. Stored inside the BAInitialDownloadRestrictions dictionary.
|BADownloadDomainAllowList|Array|Array of domains that can assets can be downloaded from prior to first app launch. Stored inside the BAInitialDownloadRestrictions dictionary.
|BAMaxInstallSize|Number|The combined size (in bytes) on disk of the Non-Essential assets that download immediately after app installation.
|BAManifestURL|String|URL of the application's manifest.

info plist requirements (new)


|Key|Type|Description|
|-|-|-|
|BAInitialDownloadRestrictions|Dictionary|The restrictions that apply to the set of assets that download prior to first app launch.
|BADownloadAllowance|Number|The combined size of the initial set of non-Essential asset downloads. Stored inside the BAInitialDownloadRestrictions dictionary.
|**BAEssentialDownloadAllowance**|Number|The combined size (in bytes) of the initial set of Essential asset downloads, including your manifest. Stored inside the BAInitialDownloadRestrictions dictionary.
|BADownloadDomainAllowList|Array|Array of domains that can assets can be downloaded from prior to first app launch. Stored inside the BAInitialDownloadRestrictions dictionary.
|BAMaxInstallSize|Number|The combined size (in bytes) on disk of the Non-Essential assets that download immediately after app installation.
|**BAEssentialMaxInstallSize**|Number|The combined size (in bytes) on disk of the Essential downloads that occur during app installation.
|BAManifestURL|String|URL of the application's manifest.



# Sample implementation

Add required key sto app info.plist
contains a background download extension
* app bundle identifier as prefix
Ensure app and extension
* share a common app group
* Signed with a team identifier

migrate to bg assets to fetch actual sessions.  

`withExclusiveControl` guarantees mutual exclusion with the other process (extension/app) to improve isolation.

promote to fg to decrease time to finish.  Great opportunity if using the app.
if doesn't exist, create.

I guess there's 1 delegate per app + extension?  So if you change it you nuke the other one?

Use move so that we're not copying, for purgeability, etc.  

Re-fetch content that's missing, etc.

Downloads can fail, due to network issue, etc.  bg assets does retry.  But after a certain point in time you want to find out.  downloads promoted to fg will fail almost instantly.  

implement app extension for bg downloading.  Provides support for enqueing essential assets.  Extension schedules downloads while your app is not running.

extension processes updates if no delegate.  

when launching withExclusiveControl we need to move file out of temporary location, since withExclusiveControl is asynchronous.

Consider re-enqueing failed downloads as non-essential.  Use `removingEssential` to make this copy.


# Debugging guidance

since app installation is controlled by appstore, need to force extension to launch in order to debug.

tool  built into xcode
`xcrun backgroundassets-debug`.  see help, manpage
simulates install, update, periodic events

multiple device support
Requires a paired iOS device
Developer Mode must be turned on

Things to remember
* essential downloads supported only during app install
* app can be launched without all essential assets
* Non-essential downloads begin after app is installed
* Extension runtime and memory usage is limited
* Make your app a good storage space citizen

# Wrap up
eliminate waiting iny our apps
adopt essential assets
verify with TestFlight
Reach out on developer forums
Give us feedback

[[Meet Background Assets]]



# Resources
* https://developer.apple.com/documentation/backgroundassets
* https://developer.apple.com/documentation/backgroundassets/downloading_essential_assets_in_the_background
* 