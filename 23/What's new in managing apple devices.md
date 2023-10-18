Learn about the latest management capabilities for iOS, iPadOS, and macOS. Discover how you can streamline the setup experience with enhancements to automated device enrollment and a new return-to-service option for iOS and iPadOS devices. We'll share how to use your identity provider in even more places on macOS and show you how Apple Configurator can help automate tasks.

# managed apple IDs
updates to continuity, apple wallet, icloud keychain, etc.

openID connect support to ASM/ABM.  This allows federation with any identity provider.

[[Do more with Managed Apple IDs]]

# passkeys

[[Deploy passkeys at work]]

# declarative device management

[[Explore advances in declarative device management]]

# apple watch
now enrolled into MDM
[[Meet device management for apple watch]]

# macOS

organizations want to ensure security settings are in place before macOS is enrolled

* filevault
* specific version
* MDM

* enforce filevault
	* enable during setup assistant
	* show recovery key or escrow to MDM
* require a minimum OS version
	* json 403 response
	* guided to update OS
	* restarts automatically
* enforced automatic device enrollment
	* this year we've changed teh UX to launch into a full-screen setup assistant.  After network is connected, user can continue enrollment or not now.  allows the user 8 hours before they're required to enroll in MDM.

## platform SSO

introduced last year?
this year, we're taking it further with exciting new capabilities.

system settings now shows status.  Reauthenticate, etc.

create local user account just-in-time
enabled by the new UseSharedDevice keys.
requires the device to be:
* online
* at login window with filevault unlocked
* managed by an MDM supporting bootstrap tokens
username and password or smartcards to create account

## smartcards
* standard, administrator, or group defined permissions
* mapping of identity provider groups
* non-local users at authorization prompts

exceptions: 
* current user
* securetoken or ownership

## password management
* password policies with regular expressions
* password compliance management
	* verify compliance after a password has been set
	* notification is shown during an active log-in session
	* password change notification prompted on next login

## system settings management

with introduction of system settings, isntead of hiding panes, administrator can restrict specific settings.

apple ID login and internet accounts
adding local-user accounts
many more, time machine,e tc.

## managed device attestation

* attested device identity
* attested device properties
* hardware-bound keys


deployment models:
* secure the MDM channel
* acme server performs trust evaluation
* relying parties perform trust evaluations

we've brought managed device attestation to macOS

* deviceinformation attestation
* acme attestation
* supports hardware-bound keys
	* stored in data protection keychain
	* VPN, 802.1x, kerberos, exchange, MDM
[[Discover Managed Device Attestation]]

new properties

even more management updates to discover.  Let's take a look at application management

## managed mac apps
* app configuration and feedback
* automatic removal on unenrollment
* remove an app via MDM

package can contain multiple applications
multiple applications are manageable
MDM can remove individual applications
content outside of /Applications is not managed

# iOS and iPadOS
## return to service
previously, getting devices back into service requires someone to touch the devices.

1.  MDM -> EraseDevice
2. new information that does everything

New `ReturntoService` dictionary
* wifi settings
* include an enrollment profile
* previous language and region settings get applied

## easy student signin

signin proximity card.  Proximity card does the apple watch pairing thing, point cloud.  Teacher scans the cloud, teacher continues onto the student selection process.  Teacher looks up student.

requirements
* teacher and student in the same ASM location
* local proximity of devices
* students have authorized the teacher on personal devices

easier to login to a shared ipad

`AwaitUserconfiguration` allows you to fully configure a device after login

`SkipLanguageAndLocaleSetupForNewUsers`.  
configure quota for temporary users on shared iPad

## cellular connectivity

iPhone and iPad support private LTE, standalone and non-standalone 5g networks.
power efficient activation of private-network SIM based on geolocation
intelligent selection between private and public-network SIMs
option to prefer cellular over wifi

## 5g network slicing
* configure managed apps to use specific 5G network slice
* slice name for `CellularSliceUUID` to be defined by the carrier
* Mutually exclusive with VPN configurations

## relay network extension

access enterprise resources without a VPN
`com.apple.relay.managed` payload type and NERelaymanager API
per-app, per domain, or default route configurations
compatible with iCloud private relay
[[Ready, set, relay Protect app traffic with network relays]]

many more new configuration possibilities.

in a future release, we will remove
* APN payloads
* toplevel cellular keys in DeviceInformation
* listed restrictions to require supervision

check out documentation for details.

exciting news about apple configurator
## apple configurator

iPhone used by many admins, etc.

1.  Device added to organization
2. add to MDM server

this year we allow you to automatically assign to MDM server right in apple configurator.

Users have 3 options:
* don't assign
* assign to default MDM server
* assign to selected MDM server


###  configurator for mac

additional shortcut actions
* update
* restore
* erase
* prepare
run shortcuts on attach or detach
leverage any shortcut action in workflows

basic shortcut for provisioning an iPad.  Reset device, install latest version of iPadOS.  Prepare.  Mac shares internet connection and runs caching server.

Use new shortcut settings in apple configurator to run shortcuts when attaching/detaching device.

## mdm integration
send commands to devices
* update inventory
* install app
* restart or shutdown
interact with the MDM server
* add device to a group
* update device attributes

# wrap up
* adopt new macOS features
* discover iOS and iPadOS features
* send us feedback
* 
# resources
* https://support.apple.com/guide/apple-configurator/
* https://support.apple.com/en-gb/guide/apple-configurator-mac/welcome/mac
* https://developer.apple.com/documentation/devicemanagement
* http://github.com/apple/device-management
* https://support.apple.com/guide/shortcuts-mac/welcome/mac
* https://support.apple.com/guide/deployment/welcome/
