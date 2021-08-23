86% of all iPhones introduced in the last 4 years, use iOS 14.

Want to ensure updates are installed as soon as possible, minimizing disruption to the user.

# Test
AppleSeed.  Get access to prerelease software for testing.

Once publically released, you may still need time to test.  Don't want to be available to all users.

Admin deferral: Prevents supervised devices from updating for some time.

## Defer
* Default delay is 30 days
* Can specify up to 90 days
* Enhanced control of updates interface
* MDM commands are not affected

* iOS 11.3+
* iPadOS 13.1+
* tvOS 12.2+
* macOS 10.13+

Same deferral window for major and minor.

For macOS 11.3+, can delay major releases longer than minor releases.
`forceDelayedMajorSoftwareUpdates`
`forceDelayedSoftwareUpdates`
`forceDelayedAppSoftwareUpdates`

# Deploy
1.  Updates were only able to evaluated on user's device
2.  Device queries apple for updates
3.  Returns data to MDM server

However, apple provides the "Apple Software Lookup Service".  You may already be using this feed.  For iOS, MDM solutions use this feed to be aware of available updates.

1.  MDM consults the service
2.  Send command to device
3.  Device installs from apple

* Unified workflow for macOS and iOS
* Query `SoftwareUpdateModelID` key

Includes appropriate hw identifier for macOS.  Determines applicability on iOS without hardware command.

Target an update by specifying `ProductVersion` key.

No more device round trip to scan for updates.

## Considerations
* Requires supervision
* iOS and iPadOS requires passcode
* Ownership for macOS

Defines who is authorized to make changes to startup security policy.  On apple silicon, policy defines restrictions around which versions of macOS can boot.

macOS authentication
* User password
* bootstrap token
## InstallLater
 Bootstrap token with InstallLater.  
 Avoid disruption
 MDM support
 ## Install updates
 * `ScheduleOSUpdate` command
	 * InstallASAP
		 * Note that this does not automatically close applications 
		 * DownloadOnly and NotifyOnly
		 * InstallLater (usually 2am-4am with ML)
	
## iOS, iPadOS
* ScheduleOSUpdate command
	* Default
	* DownloadOnly

Password-protected devices requires password.  So we prompt.

Later shows the passcode prompt.

MDM can flag an update as 'required'.  If so, user can say "Remind me later" up to 3x.


# Enforce
After update deferral period is over, want to ensure updates get installed.

* InstallLater
	* Specify maximum number of deferrals with `MaxUserDeferrals`
	* Implies `InstallForceRestart` when max deferrals are reached
	* Notification informs user of remaining deferral count
	* Send a new command to modify

# Update options
iOS now offers two update options

`RecommendationCadence`
* 0 => default view.  Major and minor updates are available.
* 1 => device shows lower version number
* 2=> device shows higher version number

Note that deferral restrictions still apply.

> The two update choices will eventually converge.

# Wrap up
* Deploy updates using OS version
* Automate `InstallLater` updates with the Bootstrap Token
* Enforce and schedule updates with `MaxUserDeferrals`
* Manage update options

* https://developer.apple.com/documentation/devicemanagement/schedule_an_os_update
* https://appleseed.apple.com/
* https://support.apple.com/guide/mdm/




