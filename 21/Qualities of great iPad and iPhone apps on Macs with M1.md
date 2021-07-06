>1M apps available on appstore.

Easiest way to the Mac.
* Unmodified app
* On the MAS
* Compatibility focused

Ensure a smooth app experience without making any changes.

Share extensions, widgets, photo extensions, AUs, and more.  Very likely that your existing iPad or iPhone app already works well on the mac out of the box.

Basic functionality, such as tex tinteractions, mac menu bar, etc.  Also a lot of advanced functionality like backgroudn app refresh, etc.

[[IPad and iPhone Apps on Apple Silicon Macs]]

[[Qualities of a great Mac Catalyst app]]
[[21/What's new in Mac Catalyst]]
# API mappings
We've bridgged them to give you the same great features on the mac.

* UIKeyCommand
* UIPress

## Menu bar
* Discoverable features and shortcuts
* Structure determined at launch

Pre-populated
* based on app features

UIResponder.keyCommands
* Not in menu bar
* take priority over menu shortcuts

UIMenuBuilder
* Semantic structure for UIKeyCommands
* Menu overlay on ipad
* Also Mac menu bar

Customizations will be reflrected in main menu.

Rely on responder chain to find an applicable target for the action. Determines whether or not an action will be enabled.

[[Take your iPad apps to the next level]]
[[Focus on iPad Keyboard Navigation]]
[[Qualities of a great Mac Catalyst app]]

## Drag and drop
* UIdragInteraction
* UIDropInteraction

## Printing
UIPrintInteractionController
* Bridged to mac print dialgoc
* UIApplicationSupportsPrintCommand

[[21/What's new in Mac Catalyst]]

Credits.rtf, credits.html, etc.

## Multi scene support

* UIApplicationSupportsMultipleScenes
* Scenes become windows
* Menu item "New"
* Window restoration
* Zero scenes possible!

## Multitasking support
* Resizable windows
* UIWindowScene.minimumSize
* UIWindowScene.maximumSize

* Do not use UIScreen size for layout

## Full screen apps
* No multitasking support, or UIRequiresFullScreen
* * Fixed scene size and aspect
* Scaled as needed
* Interface orientations


# Best practices
## Write portable code
Only use officially-uspported APIS
Do not hardcode paths
View hierarchies may be different

## Camera properties
* Use AVCaptureDeviceDiscoverySession
* Handle any possible configuration

## Unavailable hardware
Handle lack of 
* ARKit
* Multi-touch / CoreMotion
* CoreLocation

ARKit is nto supported.  You have made it a required device capability so it won't appear on MAS.

Only show AR features in your UI on devices that have this capability.

Offering alternatives that are more suitable to mac trackpad.  Touch alternatives may be able to help in this case.

Using CL, your app should remain usable even if no accurate location data is available.

Offering manual location entry as an alterantive.

## Recent improvement

Better window sizing (11.3)

We pick the largest supported device size that fits the screen.  From your app's POV, device size is fixed for the session.  Window makes use of available space.

Window contents are scaled up while maintaining original scene aspect ratio.

If window is moved to smaller screen, will reduce scale.

use window zoom to toggle between two zoom factors.  Priotizing the natural size of UI elements, and prioritizing pixel-perfect accuracy.

Switch between 2 zoom factors.

Virtual game controller (macOS 11.3)
Use the mac's keyboard/trackpad as a virtual controller.

Sensitivity slider, pointer hiding

[[Advancements in Game Controllers]]

Better touch alternatives
* Tilt (macOS 11.3)
* improved preference UI
* Auto opt-in (optional)
* Onboard dialog

Virtually tilt the device.  A number of additional games.

Graphical representation in preference panel.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>defaultEnablement</key>
    <true/>
    <key>version</key>
    <real>1</real>
    <key>requiredOnboarding</key>
    <array>                              <!-- Only include applicable features! -->
        <string>Tap</string>
        <string>Arrow Swipe</string>
        <string>Scroll Drag</string>
        <string>Tilt</string>
        <string>Trackpad Capture</string>
    </array>
</dict>
</plist>
```

Include only features useful in your app.  Only those features are onboarded.

Onboarding dialog shows keyboard controls.

Take window fullscreen.  Contents are scaled up to fit screen.

On iPad, device is tilted to steer.  On mac, wasd simulates tilting.  Trackpad can click button on the screen.

## Applepay
available on m1 macs, same APIs.

* Support for apple pay
* Must implement

```swift
optional func paymentAuthorizationController(_ controller: PKPaymentAuthorizationController,
	didRequestMerchantSessionUpdate
	handler: @escaping (PKPaymentRequestMerchantSessionUpdate) -> Void)
```

[[What's new in Wallet and Apple Pay]]

## avkit
* better full screen video
	* AVPlayer
	* AVPlayerViewController
	* HDR
	* mac UI

Make full use of mac display as appropriate for video content.

Add new APIs to aVPlayerViewDelegate.

AVFoundation supports HDR playback and streaming on macs with M1.  No mac-specific adoption work is needed.

AVKit controls in iPhone/iPad apps look the same as mac apps.  Full advantage of mac trackpad with support for new gestures

[[What's new in AVKit]]

## Support for shortcuts

[[Meet shortcuts for macOS]]
[[Design great actions for Shortcuts, Siri, and Suggestions]]

# Mac deployment

Door lock might be useful for example.  So reconsider opting out.

## Opting back in
* make this app available

Automatically pick the best macos version.

ASC => Can choose minimum macOS version

`LSMinimumSystemVersion` (macOS)
`MinimumOSVersion`

## Local testing
* Familiar workflows
* Different run destination

We've added testflight support.  iPhone, Ipad apps.

* Distribute to macOS beta testers

[[Meet Testflight on Mac]]

# Welcome to macintosh
Veirfy and opt in
Better on iPad and iPHone => better on mac



Your video app may work fine on big sur, but maybe you want to restrict to monterey.  Two options. 




# Recent improvements
# Mac deployment
