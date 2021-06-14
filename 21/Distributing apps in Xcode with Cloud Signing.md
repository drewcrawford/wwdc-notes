#xcodecloud #xcode 

# Xcode to app store connect
Cloud signing.

Upload vs export.  Only difference is that xcode won't upload the IPA.  Submit later via transporter.

New in XC13, prompt to create an app record.  

## Cloud signing
I need to ensure that I have a cert and private key installed on my key, which often requires manual setup.  In xc13, no certificate setup is required.  

Xcode will take the app and generate a partial signature.  This partial signature cnotains hashes of the content within your app.

Those hashes are set to the server, they sign the app in the cloud.  Signature is returned to xcode.

Anyone with an admin or account holder role can cloudsign for distribution by default.  Can also allow Developer role by granting permission on ASC.

## Build numbers
Distribution workflow can now "Manage Version and Build Number".  If xcode detects that I"m distributing with a build number that is already used, it will offer to increment to a valid number.

This cycle will continue until I have a build that meets the desired quality.

# other distribution options
Distribute via MAS.  Mac app will use the same app record as iOS version.  

Use Testflight across all apple platforms.  

Distribute via Developer ID.  
[[all about notarization]]

## Ad hoc
Up to 100 devices registered to my team.  
## Enterprise
Devices in company.  Xcode provides 2 options.
* Custom apps => via ASC.
* Enterprise distributions => 

[[Custom app distribution with Apple Business Manager]]
# automating distribution
[[Explore Xcode Cloud Workflows]]

```bash
xcodebuild -exportArchive
-archiveBuild path/to/build
-exportOptionsPlist -> optiosn in distribution workflow.  If you export an IPA, this is created.
```

Two ways to have valid credentials

* Sign into xcode prior to running xcodebuild.  Session will persist on my machine "for a period of time"
	* `-allowProvisioningUpdates` allows xcodebuild to communicate with apple servers.
* new in XC13, can use appstoreconnect keys (API keys) to sign into xcodebuild, without signing into xcode.  Can be passed to xcodebuild.
	* issuer ID `-authenticationKeyIssuerID`
	* key ID `-authenticationKeyID`
	* api key `authenticationKeyPath`

Still requires `-allowProvisioningUpdates`.  

 ```bash
 xcodebuild -exportArchive
 -archivePath path
 -exportOptionsPlist plistpath
 -authenticationKeyIssuerID ...
 -authenticationKeyID ...
 -uathenticationKeyPath path
 -allowProvisioningUpdates
 ```
 [[Explore Xcode Cloud Workflows]]
 [[App distribution - from ad-hoc to enterprise - 19]]
 