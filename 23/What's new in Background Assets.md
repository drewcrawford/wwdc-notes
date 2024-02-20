Waiting is no fun! Discover how Background Assets can help your app download content before it even launches. We'll show you how to integrate Background Assets into an existing app, explore when to use essential or non-essential assets, and learn how to make debugging your extension a breeze.

info plist requirements (old)

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


# Resources
* https://developer.apple.com/documentation/backgroundassets
* https://developer.apple.com/documentation/backgroundassets/downloading_essential_assets_in_the_background
* 