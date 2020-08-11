# Automated Device Enrollment
Requires apple school manager or apple business manager
Automatically enroll devices in MDM

# Auto advance for Mac
* Power and ethernet required
* Requires using ABM or ASM
* Auto-advance setup assistant
* Land at login
* Encrypted volumes require passwords

# Lights Out Management for Mac Pro
Remotely startup, shutdown and reboot
MDM server and a MEM-enrolled LOM controller mac

Basically an LOM controller is a designated mac that can send reboot etc. commands to mac pros.

# Ongoing management?

Any mac enrolled in a user-approved MDM will now be considered Supervised.
Control activation lock, leverage bootstrap token
query list and delete local users
Replace or remove profiles and install supervised restrictions via MDM.

## Managed software update
* Force software update
* Defer major releases for 90 days
* Defer non-os updates
* Removal of software update atalog
* Removal of ignore flag

## Managed apps
## Caching
Now handles internet recovery image

# Security
## Bootstrap tokens
Reserved encyrption key provided by MDM server enabling macos to create admin accounts.

Historically, admins needed complicated workflows to add invidiaul user accounts.

Bootstrap tokens ...
* Enable users to get SecureToken
* Supported on latest macs with apple t2
* Authorize software updates and kernel extension


## Prevent accidental profile installs
In iOS and iPad OS, we download a profile and put it in settings.

We do the same thing on the mac now.

Remains in system preferences for 8 minutes.

As of macos big sur, you can't install profiles using terminals.  It will treat it as if it's downloaded, and you'll have to complete download in the system preferences.

## networksetup

Now users and admins have different abilities.  We've also hardened security by honoring the setting that uses your admin password 

Standard users can only read network settings, turn wi-fi- password on-off, change access point.
Harden security by enabling checkbox to require password.
Override with `sudo`

# Serial numbers
Contain identifiable information, such as where and when a product was manufactured.

Apple will start returning completely random serial numbers.
Current products use existing format and new products may use the new format.

New MDM commands.

# Apple configurator
Now supports apps and books locations that are provided by ASM/ABM.
Can now choose the 'location', like middle school or a highschool

# iOS deployment
Customize setup assistant
New skip keys

Setup Assistant payload now on iOS.

# Shared iPad
Now supported for Business!

Sign in with Managed Apple ID created using ABM
Microsoft Azure active directory support with federated ID

Dynamic numbers of cached users
Delete all users at once
New queries for Estimated Resident Users and Quota Size

Temporary session – allows a guest account

# Non-removable managed apps
Previously, we let the admins lock the homescreen but that was annoying.

Also prevents offloading
# Data security

## Shortcuts
Managed Open In.  Prevent data flow between managed and unmanaged apps and services.  If it is not allowed, it will deny your ability to run the shortcut.

## Notification previews
`never` `always` `onlyAfterUnlocked`

Only respected on supervised devices.

# Set timezone MDM command
Choose the timezone for each device
doesn't require location services

# Security
## VPN
* Full tunnel (all traffic)
* split tunnel
* per-app vpn
* account

### Per-account VPN for iOS
Replacement VPN for contacts, calendars, and mail domains
Associate individual account with VPN
Replaces keys in the domain payload

## Encrypted DNS
Configure encrypted DNS without configuring a VPN
Supports Network extensions

## Mac address randomization
Can be disabled in payload.



