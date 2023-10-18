#privacy 

At Apple, we believe that privacy is a fundamental human right. Learn about new technologies on Apple platforms that make it easier for you to implement essential privacy patterns that build customer trust in your app. Discover privacy improvements for Apple's platforms, as well as a study of how privacy shaped the software architecture and design for the input model on visionOS.

Privacy pillars
* data minimization
* on-device processing
* transparency and control - help people understand what, why, when, where.
* security protections - strong technical mitigations to enforce the other pillars.  e2e, etc.

# New tools
Make it easier for you to build great apps with great privacy.
* embedded photos picker
* screen capture picker
* write-only camera access
* oblivious HTTP
* communication safety

## photos picker

photo libraries have grown massively.
A fundamental way to build trust in your app is to empower people to build fine-grained decisions.  If someone wants to use your app to share their most scenic photos from your last trip, they can do so.

New API to give you access to selectedp hotos/videos without requiring permissiont o access the entire photo library.  Now you can embed the picker into your app.  Even though thep hotos look like a part of your app, they are rendered by the system and only shared when selecting.

You can choose whether to show the picker without any chrome, or as a minimal single roll of photos.  Or as an inline version of the picket with a new options menu, for photo metadata options.

* use photos picker when possible
* no separate permission required
[[Embed the Photos Picker in your app]]

iOS 17 has a redesigned permission dialog that includes the number of photos and a sample of what will be shared.  Helps people make the decision that is right for them.  Because the preferences might change over time, the system periodically reminds people if the pap has full access.

## screen capture picker
Share only the windows/screen that your app has while screensharing.  

macOS only shares the selected window/screen.  Because of the explicit action to record the screen, your app is permitted to record the screen only for the duration of the session.  So you don't need permission to record the whole screen, it's handled by SCContentShare?Picker.

Also allows quickly adding/removing screen content to the capture session, or ending it altogether.

SCContentSharingPicker
* customizable
* No separate permission required
[[What's new in ScreenCaptureKit]]

## calendar

for apps that are only seeking access to write events, this can result in a failure to add events, frustration or maybe even people missing a concert, etc.

two important changes to apple's platforms for calendar access.  First, if your app only creates new calendar events...
* no permission needed to create new events - for EventKitUI
* VCs are rendered by the system

If you prefer to provide your own UI, new write-only permission.  Add events without access to "other events in the calendar"?

And, should your app need full calendar access later, you can ask once to upgrade.  When linked to user intent.  

Previously granted Calendar permission defualts to write-only
Apps that have not updated to the latest EventKid SDK:
* prompts defualts to write-only access
* implicit upgrade when app fetches events
[[Discover Calendar and EventKit]]

## oblivious HTTP
cellular/wifi network operators can observe what servers someone connects to.  Observe personal app usage and life patterns.  Some network operators might want to use their position to learn user data.

IP addresses are an essential element.  However, the IP can be abused to determine someone's location or identity.  Exposure to IP addresses can create various challenges.

OHTTP relay.  In order to help you protect people's app usage and IP address, by separating who from what.  Standardized internet protocol that is designed to be lightweight, proxying encrypted messages at the application layer.

Network provider can only observe connection to OHTTP proxy ("relay").
Final connection to your application server is made by the relay.  No single-party has full visibility of the source IP, destination IP, content.  Enables you to add technical guarantees to features where you don't want to be able to identify or track users.

[[Ready, set, relay Protect app traffic with network relays]]

Replace IP address with alternatives
* private access tokens
* encrypt DNS
[[Replace CAPTCHAs with Private Access Tokens]]
[[Enable Encypted DNS]]

## communication safety

* airdrop
* facetime
* phone app sharing
* photos picker

we also make these features available to everyone with sensitive content warning.
Now, with new analysis framework, you can detect sensitive content.  On-device processing.  System-provided ML models

```swift
// Analyzing photos

let analyzer = SCSensitivityAnalyzer()
let policy = analyzer.analysisPolicy

let result = try await analyzer.analyzeImage(at: url)
let result = try await analyzer.analyzeImage(image.cgImage!)

// Analyzing videos
let handler = analyzer.videoAnalysis(forFileAt: url)
let result = try await handler.hasSensitiveContent()

if result.isSensitive {
    intervene(policy)
}
```

Provide your own intervention
* obfuscate content
* Option to view content
* Adjust intervention based on enabled feature
* see docs


# Platform changes
* mac app data protection
* advanced data protection
* safari private browsing
* safari app extensions

## mac app data protection

locations such as desktop, documents, downloads, have a system-managed permission.  This ensures people are in control over when apps have access to their private data.  For files that people directly interact with such as a presentation, spreadsheet, etc.

Might store private data in different locations, such as a messaging app, a notes app, etc.  
adopt app sandbox - see documentation
then all files created by your app, will be protected.  Apps already using sandbox, get this protection automatically.

Second, if your app accesses data from other apps, a few ways to ask for permission.

sonoma will ask for permisison when your app accesses a file in another app's data container.  Permission is valid for as long as the app remains open.  When the app is quit, this resets.  If the timing is surprising or the purpose is unclear, your app's access might be denied.  Choose useful purpose strings.

alternatives:
* NSOpenPanel
* full disk access
* all apps signed with your team ID can access data in your other containers by default.
	* you can define a more restrictive policy.  ex if you build a code interpreter, maybe you want to avoid accesing your own apps.  `NSDataccessSecurityPolicy` to replace the default "same team" policy with an explicit allow list.
## Advanced data protection
follow a few steps:

use encrypted data types
* CKAsset
* Encrypted variants: String->EncryptedString instead.

Then use the `.encryptedValues` API to retrieve/store data in cloudkit.  
Your apps' data receives the full benefit of advanced data protection.

[[What's new in CloudKit]]

## Safari private browsing
two new protections.

* known tracking/fingerprinting resources from being loaded.

consider testing:
* logins
* navigation
* 2d canvas API/webGL
* apis exposing screen geometry
* web audio API

see web inspector.  Network requests that were blocked as a result to contacting known trackers, will see "Blocked connection to known tracker".

another new protection is removal of tracking parameter as part of browsing navigation or when copying a link.  Safari strips the identifying part of the URL, while leaving nonidentifiable parts intact.  Remember that app attribution can be done without identifying individuals across website

[[Meet privacy-preserving ad attribution]]

Kinda puzzled about this though, can't you just merge everything into an opaque queryparam?



## safari app extensions

With safari 17, apple permissions is coming to app extensions as well.
* per-site permissions
* User controls whether extension can run in private browsing mode
[[What's new in Safari extensions]]



# Spatial input model

There are no new permissions, no work for app developers, and no worrying about apps tracking where you look.

Interface goals
* fast and natural interaction
* Users know where they tap
* iOS and iPadOS apps work seamlessly
* Apps do not need new permissions

Privacy goals
* verify limited access to eye cameras
* apps work without eye and hand data
* Apps do not know what users look at

eye and hand input is processed in an isolated system process.  Delivered to eye/pinch detection systems.  Abstracts complex processing away from your app.

Hover feedback combines what is shown on screen with eye position to determine what the user is looking at.  If they are looking at a UI element, system adds a highlighting layer.

**Outside of the app's process, only visible to the person using the device, without revealing any information to apps.**

As soon as a pinch gesture is detected, system delivers a normal tap event to your app.  With this architecture, the complexity of translating camera data into input events is handled by the OS.  No changes are required to receive inputs on this new platform.

* type
* shape
* affected elements

[[Meet SwiftuI for spatial computing]]
[[Elevate your windowed app for spatial computing]]

**Great features and great privacy**.

# Next steps
* access data through content pickers
* Adopt Sensitive Content Analysis
* Sandbox your macOS app


# Resources
* https://developer.apple.com/documentation/security/app_sandbox
* https://developer.apple.com/documentation/sensitivecontentanalysis/detecting_nudity_in_media_and_providing_intervention_options
