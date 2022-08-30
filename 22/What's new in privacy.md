Privacy is a fundamental human right.  Center of everything we do.

Privacy is how you maintain user's trust.  When peopel understand what data is collected and how it's used, they're more likely to fully engage in your app or service.  At apple, we've prioritized great features and great privacy.  Help you deliver this to your customers.

pillars
* data minimization
	* Use only the data needed to build the feature
* on-device processing
	* If feature requires sensitive data, do it on device
* transparency and control
	If sensitive data is sent off-device, make sure people understand what is sent, how it's used, and give them control
* Security protections
	* Protect sensitive data in transit and at rest, both on and off device.

# Platform updates
Several important changes.
1.  Device name entitlement.  REstircts access to device name
2. Location indicator now shows app attribution in control center
3. improvements to gatekeeper that verify integrity of notarized apps in more places
4. launching mac apps at login now notifies people
5. pasteboard access now requires permission

## Device name entitlement
The user's name from their apple ID account is included by default as part of the device name.  UIDevice API allowed apps to access.  To better align with user expectations, the UIDevice.name API will return the model of the device rather than the user-assigned name, regardless of how people customize it.

Some apps have multi-device experiences that utilize the device name.  
Entitlement available if
* multi-device feature
* device name visible to user
`com.apple.developer.device-information.user-assigned-device-name`

Sharing with third parties other than cloud hosting providers is **not permitted**.

## Location
Swiping down from control center indicates which app is using location.  Make sure that your app only uses location when expected to avoid surprising people.

## Gatekeeper improvements
In macOS ventura, gatekeeper will now check integrity of all notarized apps, not just quaratined apps.

Apps must be properly signed.  If your notarized app is no longer properly signed, it will be blocked by gatekeeper on first launch.

In addition, gatekeepr will also prevent your app from being modified in certain ways.  Apps validly signed by the same developer account or team will continue being able to update each other.

To allow another team to update your app, or restrict apps to your updater, can update your info.plist.  Here unrelated app can allow another app to update.

Simply add
`NSUpdateSecurityPolicy` `AllowProcesses`.  Dictionary mapping team identifier to an array of signing identifiers.  

If an app is modified by something that isn't signed, and it isn't allowed by NSUPdateSecurityPolicy, macOS will block the modification and alert the user.  Clicking on the notification sends them to system settings where they can allow.

review
* ensure apps are validly signed
* adopt an NSUpdateSecurityPolicy (if needed)
* Users need to allow out of policy updates

## Launching mac apps at login
> Sometimes when a user logs in, apps that aren't relevant may be open

what to use to launch?  demons? a gents? service management?

with macos ventura, this is simpler.  New, single API.  Allowed to launch at login by default.  Users will be notified.  Elevated permissions require admin approval.

clicking on notification opens the preference pane.  top section is apps, bottom is "other items".  agents, demons, SMLoginItems, etc.  

### Service management framework
`SMAppServiceAPI`
* launch at login from app logic
* no need for installers
* no need for complex cleanup
* works in MAS apps

## Pasteboard access
Previously, a transparency notice let people know when apps access the pasteboard when they had not pressed paste.  In iOS 16, system confirms intent for stuff between apps.  If you continue to access with UIPasteboard, the system will display a modal prompt.

How to paste without prompts?
1.  Edit options menu
2. Keyboard shortcut
3. Adopt a UIKit paste control!


# Features to adopt
## UIKit paste controls
Adopting these controls allows pasting without an edit menu, keyboard shortcut or system prompt.  System confirms intent by verifying the button was visibly displayed and tapped.

Can customize to suit your user interface.  But be sure your button has enough contrast and isn't hidden behind other elements, or it won't work. Make sure to test your paste buttons.


## Media discovery service

Wide range of streaming protocols.  To stream media using one of those protocols before, apps needed permission to access local network, or bluetooth.  This is needed because knowledge of all devices is required to manage device selection.  But this poses a fingerprinting risk.

Your app can stream to selected devices without having to prevent network or bluetooth prompts.  Same prompt as airplay.  Apps only see the device picked to stream to.

Extension can search for local network and bluetooth devices, but is sandboxed separately and **can't send results back**.  No broad permission is needed because the app cannot see the network.  Extension can onyl send discovered accessories to the `DeviceDiscoveryExtension` framework.  Presents a list of discovered deivces in the picker and after a selection is made.

Streaming protocol providers:
* Create extension with `DevicesDiscoveryExtension`
* Extend `AVRoutePickerView` entry point
* Use selected devices in media protocol
* Download sample extension and app

Application developers:
* Adopt streaming provider's media device discovery enabled SDK

Build great featuers with great privacy.  

## PHPicker expansion
Now on the mac.  Now on watchOS.  Update your mac and watch apps to access photos without prompting access to all photos.

## Private access tokens
Prevent fraud.

* Replace CAPTCHAs
* apple doesn't learn servers visited
* Device identities remain private
* Privacy pass IETF standard
[[Replace CAPTCHAs with Private Access Tokens]]

Same technology we use to identify private relay users.

## Passkeys
passwords are at the center of authenticating accounts and keeping data secure.  But hard to remember, so people make them simpelr or reuse.  Phishing.

More robust solution using the same instantly familiar UI style and biometric verification as password autofill.

* eliminate server password compromises
* prevent phishing
* sync across your devices

[[Meet passkeys]]

# Safety check
Designed to help people in domestic or intimate partner violence situations review or reset access.  SC helps in these ways
* stop sharing data with people
	* find my, photos, notes, calendar
* stop sharing data with apps
	* resetting system privacy permission for all apps
* sign out of facetime/iMessage on other devices
* sign out of other iCloud devices
* Change passwords
* Review trusted phone numbers for 2fa
* Manage emergency contacts

These help people with threats to their personal safety control access to their data.  Two ways to use safetycheck.

1.  Emergency reset.  Immediately reset access across people and apps.
2. manage sharing and access.  More fine-grained control over each capability of safety check.

Quick Exit => quickly lets people exit the flow if they're concerned someone might see what they're doing.  The next time they enter settings, they'll be back on the main settings page.

SC helps people in domestic or intimate partner violence situations take back control of their private data.  Not just about making a decision in the moment you share data but also being able to understand and change that decision at any time.

# Next steps
* Prepare for new platform updates
* Adopt media device discovery extensions
* UIKit paste ocntrols
* Private accesstokens
* passkeys




