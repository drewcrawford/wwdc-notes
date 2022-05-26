[[21/What's new in managing Apple devices]]

# MDM
* Apple-defined framework
* Enables third-party solutions
* Vary in interface and market focus

Device ownership
* organization-owned
* user-owned

Connectivity options
* wi-fi connectivity
* cellular connectivity

|                                                     | physical sim | esim |
| --------------------------------------------------- | ------------ | ---- |
| Use MDM to prevent removal                          | no           | yes  |
| Mass installed in supervised devices using MDM      | no           | yes  |
| Easily switch from one carrier to another using MDM | no           | yes  |
| Ideal for mass deployment                           | no           | yes  |


dual line - physical sim + esim
now we support *dual* esim.

## Organizaiton-owned
### Enrollment
automated device enrollment.  zero-touch deployment workflows
automaticlaly supervised
enrollment customization
modenr authentication
skip setup assistant steps
prevent unenrollment

ABM/ASM.  
Devices outside can be added to ABM/ASM with apple configurator.  30-day provisional period.
Device enrollment.  

Device enrollment - not as good
### Ongoing management
* Simple
* Secure
* etc.

* Payloads
* restrictions
* tasks
* queries

Many payloads supported.

Restrictions examples
Prevent device name change, disable icloud, app store, etc.
Managed pasteboard

private relay
Included with iCloud+.  
supervised restriction
mask-h2.icloud.com

USB pairing
`allowHostPairing` restriction => only devices with same superivision can pair

Ethernet adapter considerations
* active before first unlock
* use `allowUSBRestrictedMode`
* passcode unlock commands without wifi or cellular

Management task examples
* lost mode
* activation lock

Managed lost mode => place in lost mode.  Lock screen with message, location, sound
Aple ID or icloud not required
Location services not required

ASM or ABM => theft deterrance
Device-based activation lock. Server-side, apple/icloud not required.  

Query examples
device name, serial number, etc.
Roaming status.
Passcode compliance

[[Manage software updates in your organization]]

[[Discover AppleSeed for IT and Managed Software Updates]]

90-day deferral window
single deferral value for all updates
deferred update are hidden in settings
deferral does not limit admin installation
Defer beta updates

Update options
* choose between the latest version of the current OS or next major OS
* Choice will be available *for a period of time*
* MDM control with `RecommendationCadence`

candence 0 => 14 or 15
1 => only update 14
2 => only 15

Any deferral policy you set is respected.

```xml
<dict>
<key>RequestType</key>
‹string>Settings</string>
<key>Settings</key>
<array>
<dict>
<key>Item</key>
‹string>SoftwareUpdateSettings</string>
<key>RecommendationCadence</key>
<integer>1</integer>
</dict>
</array>
</dict>
```

### shared ipad configuration
Use SIS sync or azure ad federation
temporary session can provide access without credentails
use content caching


### Content
Apps and books to employees/students
iOS and iPadOS apps from the app store
books from apple books
custom apps

Use device-based app distribution
* deploy apps directly to devices with apps and books mdm
* does not require apple id or invitation
* retain full ownership of apps
* assign, revoke, reassign

utilize content caching
built into macOS
limit number of preinstalled apps
provide additional content on-demand
For shared iPad, pre-install all apps and show or hide apps per user or group

### Re-deployment

* erase all content and settings
* renders all data on the device cryptographically inaccessible
* apple configurator 2, finder, or MDM command

## User-owned
User enrollment.  BYOD.  

[[Discover account-driven User Enrollment]]

Requires managed apple id to establish identity on the device
cryptographically separated apfs volume for managed data
account-driven onboarding initiated in settings

Used when you cannot provide managed apple id or are outside of ASM/ABM
Less user privacy than user enrollment

### Ongoing management
Configure a corporate account and VPN settings
Require a 6-digit non-simple passcode
Deploy organization-woned apps

Can only access MDM stuff.  

lost/stolen Requires personal apple ID
User-controlled find my
User-controleld activation lock

### Content
Purchase licenses
Assign licenses to managed apple IDs
No additional T&C prompts
Retain full ownership of licenses

"Required app" can install a device
Prevent user from removing a managed app
Alerts user when attempting to remove
Ideal for deploment-critical apps

### Redeployment
Unenroll from MDM
Erase all content and settings, apple configuration 2 or finder
device migration => set up MDM as new

appleseed.apple.com/IT

# Resources
* https://www.apple.com/support/professional/
* https://support.apple.com/guide/security/welcome/web
* https://support.apple.com/guide/apple-school-manager/
* https://support.apple.com/guide/apple-business-manager/
* https://support.apple.com/guide/deployment/
* 
