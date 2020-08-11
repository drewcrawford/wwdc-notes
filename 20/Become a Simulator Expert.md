How the simulator can help you turbo-charge your development experience and create great apps

# Overview
simulator lets you test your apps for iOS, iPadOS, tvOS, and watchOS on your mac.  Simulator is built into Xcode.  etc.

# Features
In Xcode 12, we've significantly improved screenshots.  New button, control click on thumbnail.  Multiple screenshots.  If you don't interact, will be saved to desktop by default.

Dragging files onto simulator causes certain events to happen.  apns file.  

Bring Simulator into full-screen mode.  Simulator and xcode, side-by-side, in full-screen mode.

# Platform features
iPadOS supports mouse pointer interactions.  Click the pointer button in the toolbar to capture the pointer.  Once pointer capture mode is enabled, mouse/trackpad now controls pointer in the iPad simulator.  Behaves just like a physical iPad.  For example, pinch gestures.

Keyboard is connected as well, so cmd-H, cmd-tab, etc. also work.  Can capture just the keyboard but not the pointer.  

simulator keeps track of which simulator is captured.  If another window gets focus, capture stops, and restarts when activated.  Handy for debugging / breakpoint stops.  Of course you can use escape to stop capturing.


Can you change the key to stop capturing, for example to bind escape key to something in your app?  Yes in simulator preferences (app preferences).

Customize lifetime of booted simulators.  By default... Personally, I've configured my preferences to leave simulators running.  Customize visual indicators.  Show/hide device masks.  

Menubar contains a variety of options.  Features contains useful options to toggle features supported by our platform.  

Open various simulators from file menu.  In xcode 12, you can create a new simulator from the (simulator) app.

# Scaling options
* Physical size -> makes device on screen be same size as it would be in the real world.  
* Point accurate -> Sizes the window so that content appears the same size on devices with different scale factors.  
	* All points on all simulators the same on your screen regardless of pixel density.
	* 3x display appears same size as 2x display
	*   Pixels may be combined and downscaled if your mac has lower pixel density than simulated device
	*   The scale is determined by the device combination
*   Pixel accurate -> One simulated pixel lights up one pixel on your mac's display
	*   This can result in very large windows on Macs with non-retina display.

# `simctl`
Pronounced like "simcuttle"
## privacy
simctl supports many of the most important privacy services.

To see the full list, visit `simctl` help pages, by running `simctl` with no args.

```sh
xcrun simctl privacy booted grant calendar com.example.MyApp

xcrun simctl privacy booted grant photos com.example.MyApp

xcrun simctl privacy booted grant contacts com.example.MyApp
```

or revoke

```sh
xcrun simctl privacy booted revoke calendar com.example.MyApp

xcrun simctl privacy booted revoke all com.example.MyApp

xcrun simctl privacy booted reset all
```

## push notifications
sample payload for simulator.
Note we need to provide a target bundle ID.

```json
{
  "Simulator Target Bundle": "com.example.MyApp",
  "aps": {
       "alert": {
           "title": "Push Notification",
           "subtitle": "New fruit smoothies are available",
           "body": "We know you'll love these delicious concoctions 🥰"
       }
   }
}
```

```bash
xcrun simctl push booted com.example.MyApp payload.json
```

If you drag-and-drop onto the device window, the bundle ID must be in json.

With simctl, bundle can be on CLI instead.  Note that CLI will override the file.

example without CLI
```sh
xcrun simctl push booted payload.json
```

## video recording
You can capture a video of your app's functionality right from CLI.

```swift
xcrun simctl io booted recordVideo video.mp4
```

`--force` lets you overwrite, not enabled by default.

record a video without device mask.
```swift
xcrun simctl io booted recordVideo --codec h264 --mask ignored video.mp4
```

Recording continues until you terminate with ctrl+c.

simctl provides accelerated hardware encoding.

Can capture from external display.  Test with Photos app which uses the external display.

```sh
xcrun simctl io booted recordVideo --display external external.mp4
```

## Status bar
Can display however you want.  e.x. change system time, battery, etc.

```bash
xcrun simctl status_bar booted override --time 12:01 --cellularBars 1 --dataNetwork 3g --wifiMode failed
```

clear overrides

```bash
xcrun simctl status_bar booted clear
```

Various more options, see help

## Keychain
Add certificate to root store

```bash
xcrun simctl keychain booted add-root-cert myCA.pem
```

Still need to add cert manually.  Settings->general->about->certificate trust settings.

Now the CA cert is trusted.

