# Privacy pillars
* On-device processing
* Data minimization
* security protections
* transparency/control

# On-device processing
## Private federated learning (PFL)
Differentially private model updates
No user data collected by server

We can build centralized models on our servers without having access to user data.

QuickType keyboard
QuickType quick reply
Hey Siri vocal classifier

Photos sharing
Dictation language models
HomeKit secure video object detection

Developers can use CoreML to run models on device

[[Use Model Deployment and Security with Core ML]]
[[Get Models on Device using Core ML Converters]]

## On-device dictation
Use dictation without sending voice 

## HomeKit face recognition
HomeKit cameras can recognize people from your photos library
Locally on your home hub

# Data minimization
## Photos
### Limited photos library
Users can give apps access to only a limited selection of photos instead of the entire photos library.

[[Handling Limited Photos Library]]

#### PHPicker
Replacement for `UIImagePickerController`
Improved with search and multi-select

[[Meet the new photos picker]]

*No prompt!*

## Location
Precise location control
Prompt for approximate location by default -> `NSLocationDefaultAccuracyReduced` in Info.plist.

Consider how your app will respond to approximate location
Respect teh user's intent
Keep as many features as possible

[[What's new in core location]]
[[designing for location privacy]]

## Contacts

Contacts in suggestions.  The keyboard will automatically populate the correct information in that name.

`emailTextField.textContentType` = `.email`

[[Autofill everywhere]]

## Data minimization
Only ask for what you need
Limited access doesn't mean limited functionality
Build trust through accessing as little data as possible

# Security
## Server name tracking

### DNS
* DNS over TLS
* DNS over HTTPS

Encrypted DNS co-exists with VPNs etc.
[[Enable encrypted DNS]]

### TLS
Encrypted SNI

[[Secure your app: Threat Modeling and anit-patterns]]

# Transparency

## In the app store
App Store policy requires a privacy policy

* What data do you collect?
* How is the data used?
* Is the data linked to a particular user or device?
* ?

**SDKs are part of your app too**

Learn about your SDK's data practices

## On the web
How ITP has been protecting you.  Now you can see known trackers.

## in apps
New transparency UI will be visible when pasteboard accessed

## recording indicator

## best practices
Request access only when your app needs it (e.g. prewarming)
Reinforce an explanation or expectation in your app
Consider SDK use of data

# Control

## networking

### local network
Apps can get information from the local network.
Accessing the local network will trigger a prompt to the user requesting permission.
Declare your bonjour services in the info.plist

Make sure to give context for your request
Prompt at the right time
provide a clear suage description

[[Support Local Network Privacy in Your App]]

### mac address
We randomized when not connected.  But when connected, network operators can aggregate multiple users.

Now, iOS will automatically manage mac addresses when joining networks.

* Per-network address
* Not linked ot identity
* generated daily
* Users in control

## nearby interaction framework
No bluetooth or network access
Session-based range information

Available in foreground.  Include a purpose string
Prompt in context.

[[Meet Nearby Interaction]]

## app clips
iOS cleans up unused app clips.

Location access - Location confirmation, without the need for full location access

[[Streamline your app clip]]
[[Design Great App Clips]]

## Safari Web Extensions
`activeTab` permission allows you to run script on the current webpage
[[Meet Safari Web Extensions]]

## Updates for #macOS 

* Bluetooth
* Limited Photos Library
* HomeKit
* Media and Apple Music
* `CNCopyCurrentNetworkInfo`

# Tracking transparency and control
App store policy requires user permission for tracking across apps and websites owned by other companies.

Your app must display this prompt, and only track users across apps and websites owned by other companies, if they press "allow tracking".

This includes
* targeted advertising
* advertising measurement
* sharing with data brokers

Counts even if it isn't tied to my name, but any user ID, idfa, device id, fingerprinted id, or profile.

User permission not required for
* Linking doen solely on user's device
* Sharing with a data broker solely for fraud detection, prevention, or security
* Exclusively on your behalf, not for the broker's purposes

Use the `AppTrackingTransparency` framework to request permission to track
Requires `NSUserTrackingUsageDescription`

Limit Ad Tracking -> IDFA will read as 0s.  This also PREVENTS apps from asking.

Call the AppTrackingTransparency framework every time your app is launched before you want to use the IDFA.
Do not cache or store the IDFA.
Think about what changes you should make to stop tracking a user if they switch tracking off.

# SKAdNetwork
Measure ocnversion without user tracking
On-device intelligence
Aggregation
Does not require tracking permission

Today, the ad networks link based on seeing the same IDFA.
[[What's new with In-App Purchase]]

