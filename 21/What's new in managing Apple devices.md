* Privacy
* Agency
* Control

User enrollment => protects the privacy of personal data for BYOD.

[[Discover account-driven user enrollment]]

[[Manage devices with Apple Configurator]]

[[Meet declarative device management]]

[[Manage software updates in your organization]]

# Features
VPN and device management are combined to provide one comprehensive place to display the device state.  Get a complete understanding of your device.

Many MDM vendors have apps.  On supervised devices, you can install without prompting.  But unsupervised devices prompt for permission.  Which the user can decline.  Wouldn't it be great if you had more control?  (No.)

Can now specify 1 app to install on unsupervised devices.  We ensure privacy is protected

Specify iTunes Store ID In MDM payload
Device or user app license required
`InstallApplication` command required
Add managed app attribute to ensure the app cannot be removed

Managed open in => Control whether data enters or leaves.  
New: managed pasteboard.  New restriction controls if paste is affected `requireManagedPasteboard`
System and third party apps require no changes
Controls data flow between unmanaged and managed apps.
Organization name can be modified using organization info settings.  

## Shared iPad
Traditionally required managed apple id.  But with temporary session, anyone can use.

Three new keys
* TemporarySessionOnly => limits managed use
* TemporarySessionTimeout
* UserSessionTimeout => logs the user out after a set amount of time

Ensure sufficient minimum time
Timer resets if button is pressed

## Remote
Connecting specific apple tvs with the remote widget on iOS devices.  In tvOS 15, new security enhancement.  Apple TV will no longer broadcast mac address in bonjour.  So now oyu have pin prompts.

To ensure minimal impact, we added a new key: `TVDeviceName`, filter device names in the remote widget.

## Varous changes.
Payload identifiers must be unique
Unsupservised devices => .. declined up to 3 times.  

Payload keys use more inclusive language.

# Mac

## System extensions
* Installing payload activate  system extension if already pending
* Removing payload deactivates extension
* `RemovableSystemExtension` => app can deactive, e.g. on uninstall.  Now admin password is required.

## Kernel extensions
`RestartDevice` => `RebuildKernelCache`
`KextPaths` to specify additional paths.  Can install and load kernel extension without app launch.
`NotifyUser` => User can perform a graceful restart.
`Kernel Extension` payload
* AllowNonAdminUserApprovals
* **User must perform restart from within system preferences, or use the notification to trigger a rebuild.**
* `SecurityInfo` command
	* `BootstrapTokenRequired...`

## iOS apps on M1
* `DeviceInformation` query
	* `SupportsiOSAppInstalls`
* New flag to indicate iPhone/iPad app. `iOSApp` set to `True`
* For in-house enterprise apps, URL points to iOS ipa, not `.pkg`
* iOS-style provisioning profiles

## apple silicon
* `DeviceLock` => administrators can send a six digit pin code, lock message, and phone number.  Mac will reboot and present.
* User cannot use the mac until the pin has been provided.
* `SetRecoveryLock` and `VerifyRecoveryLock` to prevent mac from booting into recovery.
	* Password can only be set and removed by MDM.  So if the user unenrolls, the recovery password will be removed.
	* Server needs to know existing password
	* Continue to use Activation Lock as recovery lock will be removed when mac is erased
* Erase all content and settings
	* Via MDM as well
	* Supported on Apple silicon and T2
	* Current system volume is preserved
	* Apple silicon security settings reset
	* `allowEraseContentAndSettings` restrictions
	
See device management.  ASM, ABM.  

# wrap up

[[Meet declarative device management]]
[[Discover account-driven user enrollment]]
[[Manage devices with Apple Configurator]]
[[Manage software updates in your organization]]
[[Improve MDM assignment of Apps and Books]]

* https://appleseed.apple.com/
* https://developer.apple.com/documentation/devicemanagement
* https://support.apple.com/guide/mdm/
* https://support.apple.com/guide/deployment-reference-macos/
* https://support.apple.com/guide/deployment-reference-ios/

