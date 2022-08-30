Our shared responsiility to enable employtees to thrive in the workforce etc.  We're contininuing to develop tools.

Many great enhancements this year.

[[Meet declarative device management]]
[[Adopt declarative device management]]

[[Explore Apple Business Essentials]]
[[Discover managed device attestation]]

# Apple configurator
In setup assistant, on amac, simply ohld your iPhone running configurator over the animation.  mac will connect to the internet and add itself to your organization.

v 1.0.1 brought new explicit support for taking advantage of existing network activity.  With iOS this year, configurator for Iphone, can add iPhoen/iPad devices.  Just like adding a mac.

Any device that reequires interactive activation - activation lock, etc.  Must be handled manually.

Devices running iOS/iPadOS 16 must be added using apple ocnfigurator for iPhone.  Let's cover identity management for business/education.  
# Identity
For federated auth, aBM now integrates into google workspace as identity provider.  Use directory sync (artist formerly known as SKIM).

If you want to control where users can sign in, allow all apps or just ones on allow list in ABM/ASM.  [[Discover sign in with Apple at Work & School]]

## OAuth support
OAuth 2 added as authorization mechanism
Support a wide variety of identity providers.
Improved security with refresh token.

## Enrollment SSO
new faster method for personal devices to complete MDM enrollment.  Extensible SSO for an organization's apps and websites.  iOS/iPadOS 16 has accountd-riven user enrollment.  
Use any SSO technology
One sign-in used for enrollment

* SSO Extension app
* MDM support
* managed apple IDs
* Enrollment sSO configuration

`X-apple-MDM-esso: https://exampel.com/enrollment-sso.json`

```json
{
  "iTunesStoreID": 12345678,
  "AssociatedDomains": [
    "betterbag.com"
  ],
  "AssociatedDomainsEnableDirectDownloads": false,
  "ConfigurationProfile": "Base64EncodedProfile"
}
```

app will be used during enrollment.

You'll need to apply a new entitlement.  But you can start developng now using test mode.  Special version of the process that allows you to test your app before it's offered on the apps tore.

## Test mode
Enable in Developer sections of settings
Configure authentication response
Begin the enrollment flow
Install SSO app via Xcode, TestFlight, etc.
Complete the enrollment

`X-Apple-MDM-ESSO-Developer` header.

response is different as well
```json
{
  "AppIDs": [
"ABC123.com.example.singlesignonapp"
  ],
  "AssociatedDomains": [
    "betterbag.com"
  ],
  "AssociatedDomainsEnableDirectDownloads": false,
  "ConfigurationProfile": "Base64EncodedProfile"
}
```

everything else remains the same

## SSO extensions
In many cases,t hese are identical credentials.  In macOS ventura, we're itnroducing platform SSO.  Enable users to sign in at the login window and then autoamtically sign into apps and websites.  Platform SSO makes tokens from the login avaialble to third-party SSO extensions and works with kerberos extension.

* Initial login with local account password
* Subsequent unlocks with identity provider password
* Authentication with password or secure enclave-backed key.
* SSO tokens retrieved and shared with extension
* Kerberos TGTS also supported
* changed password validated on unlock

* Integrated sign-on experience
* Modern replacement for binding and mobile accounts
* Identity provider used only on unlock or SSO token retrieval
* Use MDM to revoke access, not this method.
## Get started
* Protocol implemented by identity provider
* SSO Extension updated by identity provider
* Extensible SSO profile updated by MDM
* configure managed devices

support.apple.com/guide/deployment

# macOS
## Software updates
Monterey introduced changes.  [[Manage software updates in your organization]]

### Deploy software updates
* acknowledge commands when in power nap mode
	* ScheduleOSUPdate, OSUPdateStatus, AVailableOSUpdate commands.
* Priority key for `ScheduleOSUpdate`.  Accepts `High` or `Low`
	* `High` mimics user initiated
	* only on minor updates

update status command
```xml
<key>DeferralsRemaining</key>
<integer>4</integer>
<key>DownloadPercentComplete</key>
<real>0.0</real>
<key>IsDownloaded</key>
<false/>
<key>MaxDeferrals</key>
<integer>5</integer>
<key>NextScheduledInstall</key>
<date>2022-04-19T09:00:00Z</date>
<key>PastNotifications</key>
<array>
	<date>2022-04-18T17:03:51Z</date>
</array>
<key>ProductKey</key>
<string>MSU_UPDATE_22A240a_full_13.0_minor</string>
<key>ProductVersion</key>
<string>16.0</string>
```

## Rapid security response
* no firmware modified
* User can remove RSR
* `allowRapidSecurityREsponseInstallation` => disables this mechanism
* `allowRapidSecurityREsponseRemoval` => prevents user from removing

## Enrollment changes
Network requirement for setup assistant "in an upcoming release"
* apple silicon or T2 security chip
* Device must be setup at least once with network
* Enforced after erase/restore

## Profiles CLI tool
Rate limiting for `show`, `renew`, `validate`
maximum 10 requests per day.
REturns cached information when exceeded.
Optional `cached` flag
See manpage for details.

## Security changes
In a future release, we're bringing certificate changes to macOS.
* No TLS Trust policy by default
* User trusts certificate manually
* Full trust for MDM Certificates
* Update workflows that involve interactive cert installation.

"Allow accessories to connect" in privacy&security settings.  Initial configuration is to ask the user to allow new thunderbolt/usb accessories.
* users consent
* Approved accessories connect up to 3 days to locked mac
* allowUSBRestrictedMode => allow wired accessories to always connect with no limitations
* Authorization for user consent
* Inherently makes user system less secure.

For other modifications, see docs.  

# iOS and iPadOS
3 main ways to manage traffic.
* VPN
* DNS proxy
* web content filter

In iOS and iPadOS 16, we're expanding per-app offerings to include DNS proxy and web content filter.  Particularly useful for user enrollment.

* secure traffic for apps
* similar to per-app VPN.

```xml
<plist version="1.0">
<dict>
	<key>PayloadContent</key>
	<array>
		<dict>
			<key>PayloadType</key>
			<string>com.apple.webcontent-filter</string>
			<key>ContentFilterUUID</key>
			<string>063D927E-ACE2-445F-8024-B355A6921F50</string>
			<key>FilterType</key>
			<string>Plugin</string>
			<key>FilterBrowsers</key>
			<true/>
			<key>FilterPackets</key>
			<false/>
			<key>FilterSockets</key>
			<true/>
			…
```

When this key is added to the payload it can only be isntalled via MDM.

InstallApplication
```xml
<plist version="1.0">
<dict>
	<key>PayloadContent</key>
	<array>
		<dict>
			<key>PayloadType</key>
			<string>com.apple.dnsProxy.managed</string>
			<key>DNSProxyUUID</key>
			<string>063D927E-ACE2-445F-8024-B355A6921F50</string>
			<key>AppBundleIdentifier</key>
			<string>com.example.myDNSProxy</string>
			…
```

```xml
<plist version="1.0">
<dict>
	<key>RequestType</key>
	<string>InstallApplication</string>
	<key>iTunesStoreID</key>
	<integer>361309726</integer>
	<key>Attributes</key>
	<dict>
		<key>ContentFilterUUID</key>
		<string>063D927E-ACE2-445F-8024-B355A6921F50</string>
       <key>DNSProxyUUID</key>
		<string>063D927E-ACE2-445F-8024-B355A6921F50</string>
	</dict>
	<key>ManagementFlags</key>
	<integer>0</integer>
</dict>
</plist>
```
* DNS Proxy and web content filter apps just work
* Secure trafficc for apps
* Similar to per-app VPN
* existing apps work without modification
* DNS proxy supports system or per-app only
* Web content filter supports seven per-app and one system-wide
## eSIM
How MDM vendors can bring all these featuers together to make for a better experience.
* provisioning new devices
* migrating carriers
* multiple carriers
* travel and roaming

`DeviceInformation` query.  

```json
{
	"CarrierSettingsVersion": "41.7.46",
	"CurrentCarrierNetwork": "CarrierNetwork",
	"CurrentMCC": "310",
	"CurrentMNC": "410",
	"EID": "89049032004008882600004821436874",
	"ICCID": "6905 4911 1205 0650 3488",
	"IMEI": "35 309418 464558 9",
	"IsDataPreferred": false,
	"IsRoaming": false,
	"IsVoicePreferred": false,
	"Label": "Secondary",
	"LabelID": "FDG4225C-L9OY-89BM-JF38-36JR4JOL76B3",
	"MEID": "35745008005631",
	"PhoneNumber": "+14152739164",
	"Slot": "CTSubscriptionSlotTwo"
}
```

Devices may return two items, asupport 2 active cellular.  EID => contains an eSIM.  Newer devices support dual active carrier ESIM.

Note that EID is unique per device.  Starting with iOS/iPadOS, other devices that return data liek this are deprecated and will no longer be returned in future of OS.

Carriers require for setup
* IMEI
* EID
* Serial
* Phone

1.  MDM uses ServiceSubscriptions query to collect data
2. Admin sends the data to the carrier
3. Data will provision eSIM on their server

installing on device
1.  Carrier will provide devices with a URL.
2. MDM sends RefreshCellularPlans to device
3. Device talks to carrier.  Installs eSIM.
4. MDM server can use ServiceSubscriptions query to confirm it was installed.

* eSIM installation is a one-time operation
	* future requests require the carrier to provision a new eSIM
* Use `allowESIMModification` to prevent users from mroeving
* `RefreshCellularPlans` works while restricted
* Set `PreserveDataPlan` to `True` when using `EraseDevice` command.

## Shared iPad
Education/business customers in a one-to-many environment.

* ManagedAppleIDDefaultDomains
	* Typing suggestion for domain name when you sign in
* Remote auth requirements
	* Currently, every 7 days
	* in IOS 16, we only use local auth
	* `OnlineAuthenticationGracePeriod` key.  Number of days between remove auth.  `0` requires all logins to be remotely authenticated
* Shared iPad quotas
	* ex ipad has 35gb
		* ResidentUsers=>3
			* effectively 11.67 gb
		* QuotaSize=>10
			* quota of 10gb
		* 3 existing users and we add 1, a user must be removed.
		* By deleting stuff, I have more space.  So now we have room for a 4th user.
		* iOS 13.4, as long as there are no chached users on the device, quotas can be adjusted at any time.

## MDM protocol and behavior changes
Accessibility.  Traditionally each user enabled themselves.  In ioS and iPadOS 16, MDM can manage.


```xml
<key>Settings</key>
<array>
	<dict>
		<key>Item</key>
		<string>AccessibilitySettings</string>
		<key>TextSize</key>
		<integer>5</integer>
		<key>VoiceOverEnabled</key>
		<true/>
		<key>ZoomEnabled</key>
		<true/>
		<key>TouchAccommodationsEnabled</key>
		<true/>
		<key>BoldTextEnabled</key>
		<true/>
		<key>ReduceMotionEnabled</key>
		<true/>
		<key>IncreaseContrastEnabled</key>
		<true/>
		<key>ReduceTransparencyEnabled</key>
		<true/>
	</dict>
</array>
```

Admins can make stuff more accessible.

* not locked after being set
* user may modify
* MDM server can also query with DeviceInformation.

InstallApplication during Setup
Install apps at `AwaitDeviceConfigured`
Very likely no user logged in => use device-based app licenses!
Unsupervised devices will still return `NotNow`

Certain datatypes are inaccessible before firstUnlock

```xml
<plist version="1.0">
<dict>
	<key>CommandUUID</key>
	<string>0001_CertificateList</string>
	<key>Status</key>
	<string>NotNow</string>
	<key>UDID</key>
	<string>00008031-000123840A45507E</string>
</dict>
</plist>
```

Ensuer you can handle NotNow here.

## Apple TV
tvOS 16 => if you erase a TV, the remote will now remain paired.  

Even more changes in the documentation.

# Documentation
Previously, we did document changes.  Reception was great.  Today we're making the sourcecode that backs this documentation publically available under MIT open source license on apple github

github.com/apple/device-management

1.  mdm documentation
	2. checkin
	3. commands
	4. profiles
3. declarative device management
	4. declarations
	5. protocol
	6. status

yaml files for each command, protocol, or declaration.

Comperhensive details about schema and structure of the repository.

* Data published back to iOS 15 and macoS Monterey
* branches for each OS releases as needed
* iOS 16 and macOS Venutra betas available now
* Build new tools or integrate into your existing product
* Share your feedback
# Wrap up
* apple configurator
* Identity
* macOS
* iOS and iPadOS
* Documentation


* https://developer.apple.com/forums/tags/wwdc2022-10045
* https://developer.apple.com/forums/create/question?&tag1=93&tag2=511030
* http://github.com/apple/device-management
* https://support.apple.com/guide/apple-configurator/
* https://support.apple.com/guide/security/welcome/web
* https://appleseed.apple.com/
* https://developer.apple.com/documentation/devicemanagement
* https://support.apple.com/guide/deployment/
