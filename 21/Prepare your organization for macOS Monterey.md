# Initial enrollment
Apple configurator for iPhone.  Assign macs to your organization.

Requires macs with T2
Adminsistrator or DEM role
Setup begins with proximinty to mac
30-day provisional period allows removal from profiles preference pane.

[[Manage devices with Apple Configurator]]


# Ongoing management
system exensions vs kernel extensions.  
* Installing payload acitvates system extension if already pending.
* Removing payload deactivates the system extension
* reboot to remove
* `RemovablySystemExtension`
	* No admin required for app to remove its own system extension(s)

Various payloads to configure additional settings and restrictions
* web content filter
* privacy preferences policy control
* notifications
* Consider compatibility for coexistence

PRK vs institutional recovery key.  IRK now legacy.

PAM modules.
* Installing or configuring requires user authorization
* Can be allowed from MDM with privacy preferences control payload
* Many MDMs already grant access 

Enterprise connect is officially sunset.  Runnig is not supported.

Preferred kerberos distribution center
restrict access to only managed apps
Allow system app access
Improved reliability for checking for required password changes
[[Leverage Enterprise Identity and Authentication]]

CLI developer utilities
* Scripting language runtiems only included for compatibility
* Won't be in further versions
* PHP removed
* Python 2 officially sunset and will prompt interactively when used
	* Note that we badge the app, which might be the MDM solution, even if it's your scripts
* `socketfilterfw` vs MDM are mutually exclusive
* MDM control of logging and logging level
* Configured logging options reported to MDM
* Options for allowing signed downloaded or system apps for incoming connections always enabled

Certificates
* Underscores in DNS now allowed
* Multiple extensions with smae id are rejected
* etc

* app data container separated
* Managed cloudkit apps will sue appleid associated with user enrollment

## networking
apple silicon supports 80211k,v,r
IPv6-only-preferredoption supported in dhcpv4 for all macs
allowCloudPrivateRelay restrictions for icloud private relay

proxy auto config (PAC)
* Http urls are deprecated
* WPAD via DNS not affected
* VPAD via DHCP option 252 is affected
* macOS may try https first
* Use https urls

## Updates
[[Manage software updates in your organization]]

`softwareupdate` tool still supported.
Required elevated privileges
Apple silicon requires volume owner credentials
Interactive prompt (default)
user and stdinpass options available
Can be used at login window
**Use MDM to deploy software updates.**

1.  Test
2.  deploy
3.  enforce

### Test
MDM restrictions to defer updates.
Defer updates up to 90 days, including betas
Separate major, minor, and non-os deferrals for macOS 11.3 or higher
Major oS upgrades as a user require administrator credentials and install assistant
Visibilitity to user vs MDM control.  MDM can still update devices if not visible to user.

### Deploy
* DownloadOnly
* NotifyOnly
* InstallASAP => default, primary.  User can cancel within 60 seconds.
* InstallForceRestart => Ignores blocking applications.
* InstallLater => Schedule update to install tonight.

InstallLater with 50% or greater battery life
Boostrap token supported for all install actions on apple silicon
User-initiated actions prompt for password
Login window and standard user support
ProductVersion and SoftwareUpdateDeviceID for updates.

### Enforce
Uses InstallLater install action
MaxUserDeferrals key
Notifies user of required managed update

By default, we still require user to confirm before installing an update, up to 5 times.

### Erase all content and settings
* Resets device to current OS version
* apple silicon and T2
* MDM EraseDevice command
* Apple silicon security settinsg reset
* allowEraseContentAndSettings restriction


MDM-initiated requirements
Booted from itnernal sealed system volume
First partition if multiple
apple silicon: bootstrap token from MDM
intel: must be in full security mode (T2)
firmware password removed (T2)

When not met, falls back to obliteration
* requires reinstallation of macos
* control with `ObliterationBehavior` option.

# Additional changes
* Shortcuts.  Future of mac automation
* Multi-year transition from Automator
* migration tool can convert most automator workflows ito shortcuts
* [[Meet Shortcuts for macOS]]

Airplay 
* share, play, present from another apple device to mac
* service in Sharing settings
* Defaults to only accepting devices with same apple id.  same network still prompts for permission
* everyone option supports peer-to-peer discovery

appls and books api
* Improved efficency for MDM
* Real-time notifications
* increased request sizes
* 25k requests can now be 10!
* [[Improve MDM assignment of Apps and Books]]


# Return to service

# Resources
* Use a managed apple id from ASM/ABM
* No invitation needed
* Detailed test plans
* Organization-focused relaase notes


[[Deploy macOS Big Sur in your organization]]


https://www.apple.com/support/professional/
https://support.apple.com/guide/security/welcome/web
https://developer.apple.com/documentation/devicemanagement
https://support.apple.com/guide/deployment/
