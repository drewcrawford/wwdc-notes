Technical deep dive of deploying macOS Big Sur.

# Platform changes
Auto-advance introduced a few years ago.  Connecting power/ethernet.

## Auto advance for Mac
* Requires ABM/ASM
* Requires power and ethernet
* Auto advances setup assistant panes
* Requires DHCP and no proxy
* Land at Login Window
* Ideal for shared or unattended use
* Assumes user accounts will be MDM or provisioned in some other way
* Fastest way to setup a mac

## Supervision for user-approved MDM
* New device enrollments are automatically supervised
* Even if the user installs themselves
* User-approved MDM is automatically converted to supervision upon upgrade
* User-approved MDM is a thing of the past

## Managed apps
* Apps can be removed by MDM command or upon enrollment
* iOS style managed app configuration and feedback are supported
* Device enrollments can convert an unmanaged app to managed

Requirements
* Install a single application into `/Applications`
* Removable using MDM from `/Applications`
* Data separation and Managed Open-In are unique to iOS/iPadOS

## Custom apps
* Now available for Mac apps
* Custom features
* Private distribution
* ABM/ASM
* Distribute to customers or internally

## Lights Out Management for Mac Pro
* remotely startup, shutdown, or reboot
* MDM and MDM enrolled controller Mac is required
* Requires controller and devices have a connection on same subnet over IPv6
* Configured using Lights Out Management payload

## Signed system volume
* Cryptographically signed system volume
* Improves macOS security
* macOS boots from snapshot
* "easier updating of the system"
* Use FileVault to encrypt user data

## Bootstrap token
Can be used on macOS Big Sur and later to grant SecureToken to any user, including local user accounts

macOS 10.15.4 and later generates one during first SecureToken enabled user login

Enables additional management features on Apple silicon

## System extensions
System extensions and DriverKit
* Endpoint security
* Networking
* File providers
* Printers/scanners

Replaces kernel extensions


Chose vendors who adopt system extensions

"We encourage you to talk to your vendors"

## Kernel extensions
Some kext categories with System Extensions equivalents do not load
Exception for MDM-allowed kexts
Loading third-party kexts requires a restart

`RestartDevice` MDM command will restart/rebuild kernel cache so user does not have to be involved. `RebuidKernelCache` tells mac to rebuild cache
`KextPaths` allows MDM to ask specific kexts to be loaded if the yhaven't been discovered by the system / found through app 
`NotifyUser` displays notification to the user and prompts them to restart.  Can be used outside of the context of kernel extensions, but especially helpful with.

`AllowNonAdminUserApprovals` key in System Policy Kernel Extensions payload.  This lets a standard user manually complete the installation of kexts.

## Downloaded profiles

User must manually approve in System Preferences
Remains in profile pane for 8 minutes
Consider in your deployment workflow

## CLI updates
`profiles` CLI tool no longer supports profile installs.
`networksetup` cli no longer allows changes to settings without admin rights
`security` tool can no longer add certs to the admin trust store without authorization

## Software update
* update mechanism similar to iOS
* Defer major upgrades and minor, security, beta, and app updates for up to 90 days
* Separate deferral windows (new)
* Force install an update and restart
* `softwareupdate` command removes `--ignore` and `--set-catalog` flags.  Use MDM.

## Privacy preferences policy control
user services vs system services

* Screen Capture and Listen Event are sensitive system-wide services
* macOS Catalina allowed standard users to approve screen capture by default.  No longer allowed
* `Authorization` key can be set to `AllowStandardUserToSetSystemService`.  After this, security/privacy settings make clear which apps can be approved.
* Existing approved entries remain approved upon upgrade

## SSO extensions
User channel profile delivery
* takes priority over system settings

Per-app VPN improvements
Calling app information
Profile removal operation

[[Leverage Enterprise Identity and Authentication]]

## Kerberos extension
* On-premise AD integration
* Password sync, automatic kerberos management
* User channel profile support
* Allow for user-level certificate identity
* Better support for per-app VPN
* More control over initial login experience
* Menu-extra represents state of extension
* Customizable UI

## Enterprise Connect
* macOS Big Sur is the last release to support Enterprise Connect
* Test and migrate your env to kerberos extension
* Migration guide available


# Apple silicon
## App compatibility
Create a single app that taps into native power/performance.
iOS and iPadOS apps on Mac
Rosetta 2
kexts must be recompiled if needed
Hypervisor framework

## Mac sharing mode
* Replaces Target Disk Mode
* SMB file share based
	* Block based transfer not available
* Uses Thunderbolt or USB cable
* Requires user authentication
* Access entire volume

## recoveryOS
* minimal macOS
* Separate hidden container
* Hold down power button at startup
* Allows reinstall of macOS
* Apple Configurator 2 can reinstall recoveryOS and optionally macOS
	* Great for rapid return to service
	* In-place macOS upgrades not supported

## login experience
Unified with and without FileVault.
* rich UI with accelerated graphics
* Supports username and password view
* SmartCard support
* Built-in CCID, and PIV compatible
* VoiceOver support

## kernel extensions and software updates
* Automatically enabled with Automated Device Enrollment
* Leverages Bootstrap Token
* Optionally enabled in recoveryOS for device enrollments to allow MDM to manage kexts
* Not necessary for MDM to do software updates

## ownership
Defines who is authorized to change security policy for a specific OS

Backed by cryptography protected in the secure enclave

Authentication required for security policy changes

Possible to be an owner but not administrator.

Installation tasks may also ask for authentication.


# Deployment workflows

## macOS installer
Eight years of hardware support in macOS Big Sur.

### acquiring and distributing the installer
* app store / system preferences
* apps and books
* `softwareupdate` command

Distribute

* remotely copy to devices
* external media?
* MDM
* Easily script the download

Fetching the installer

`softwareupdate --fetch-full-installer` Downloads latest full installer into `/Applications`

`--full-installer-version <version>` to get specific version

`--list-full-installers` to see list of available installers.  Customized for the specific mac.

Enables completely automated workflows with no pre-work required by IT.

## starting installation of macOS
### `startosinstall`
In installer's `Contents/Resources` folder
Scriptable with `--agreetolicense`
`--eraseinstall` -> reinstalls macos to default state.
* requires APFS formatted drive
* removes all other volumes in the same container.
* Replaces current OS with a default macOS

`--installpackage` -> install any number of packages after macOS finishes.
* executes in the order specified to command
* on apple silicon, requires admin/owner

## `createinstallmedia`
In installer's `Contents/Resources` folder
Minimal network impact during installation
Nearly all the files necessary are located on the external media
`--downloadassets` flag proactively downloads/stages apple t2 firmware
Internet connection is still required during install, even when using this flag.  OS on t2 needs to be personalized for hardware during installation which requires apple connectivity
External boot is disabled by default on mac computers with apple t2 security chip

## Using recovery
* macOS Recovery for Intel
	* cmd-r -> install macOS version on your mac
* Internet Recovery for Intel
	* option-command-r -> installs latest available version, may vary with t2 firmware
* recoveryOS for Apple Silicon
	* apple configurator 2
	* hold power button

## Install assistant
Users can acquire themselves
Same experience as consumers
Graphical walkthrough of installation steps
Requires admin privileges to begin installation

# Initial deployment
## 1:1 devices
Leverage automated device enrollment
* ensures supervision
* kernel extensions and software updates on apple silicon
* MDM customizes setup exerience, isntalls configurations and software
* Devices can be held in setup assistant
* Use local accounts whenever possible

[[Leverage Enterprise Identity and Authentication]]

## shared environments
Leverage ADE
* ensures supervision
* kernel extensions and software updates on apple silicon
* use auto-advance for zero-touch setup
* directory binding or identity provider helper tools provide user acounts
* Consider account expiration strategy
	* mobile accounts can auto-expire
# Upgrade
Up to you to decide which option works best in your environment.


# Return to service

`startosinstall --eraseinstall` -> does everything
Internet Recovery for Intel
* erase internal storage first

recovery or external media for intel/apple silicon
* erase internal storage first

startosinstall from recovery environment is not recommended/supported

Apple Configurator 2 for Apple silicon

# Resources



