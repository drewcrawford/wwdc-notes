# Game controller recap

* Controllers, keyboard, mouse and trackpad
* Abstracts hardware and input types
* Simplifies code
* Enables input remapping
* Consistent UI

```swift
func setupGameController() {
    // Add handler for when controller connects.
    NotificationCenter.default.addObserver(
        forName: NSNotification.Name.GCControllerDidConnect,
        object: nil,
        queue: nil)
        { (note) in
            guard let _controller = note.object? as GCController else {
              return
            }
            // Add controller input change handlers.
            _controller.physicalInputProfile[GCInputButtonA]?.valueChangedHandler = { ... }
            _controller.physicalInputProfile[GCInputButtonB]?.valueChangedHandler = { ... }
        }

    // Add handler for when controller disconnects.
    NotificationCenter.default.addObserver(
        forName: NSNotification.Name.GCControllerDidDisconnect,
        object: nil,
        queue: nil)
        { (note) in ... }
}

// Polling for controller input.
func checkInput() {
    if controller.physicalInputProfile[GCInputButtonA]?.pressed { ... }
    if controller.physicalInputProfile[GCInputButtonB]?.pressed { ... }
}
```


Controller, keyboard, pointing device.

We want to help people find games that support game controllers.  We badge them in appstore.

Curate and promote apps that work great for game controllers.  Discover apps that make game controllers shine.

1.  Add GameControllers capability
2.  no step 2

Per-app customization of game controller preferences.

Great way to improve your game's accessibility.  When you tag your game, players can customize it specifically.

## Glyph best practices
Show the correct gylph.

Consider optionally representing positional buttons.

Also buttons can be remapped.  `gamepad.buttonA.sfSymbolsName` and use `UIImage(systemName:)`.

[[Advancements in Game Controllers]]
[[Bring keyboard and mouse gaming to iPad]]
[[Supporting new game controllers - 19]]
[[Designing for game controllers - 14]]

# New virtual controller
Compatible with existing game controller input.

* On-screen touch controls

Appear like GC framework controller.  

Customizable.  Your own symbols, integrate with colors/style, etc.

They adjust to a variety of layouts depending on whether you want 1, 3, d-pad, etc.

## Virtual on-screen controller
* Left and right regions are independently configured
* layout is determined base on config
* Per-side up to 4 buttons and thumbstick, d-pad, or touchpad
* Haptic feedback is available
* Configurations cannot be dynamically modified

1.  `GCVirtualControllerConfiguration`
2.  `GCVirtualController`
3.  Do configuration on elements?
4.  `.connect()`
5.  GCControllerDidConnectNotification
6.  `GCController`

```swift
// Creating an on-screen controller

let virtualConfiguration = GCVirtualControllerConfiguration()

virtualConfiguration.elements = [GCInputLeftThumbstick,
                                 GCInputRightThumbstick,
                                 GCInputButtonA,
                                 GCInputButtonB]

let virtualController = GCVirtualController(configuration: virtualConfiguration)

virtualController.connect()
```

> So that was easy.

## Customizing buttons
Add a bezier path to buttons.
```swift
// Creating customized buttons

vc.changeElement(GCInputButtonA) { 
  config in
    let spinningPath = UIBezierPath()
    // load or draw the spinning attack path
    config.path = spinningPath
    return config
}

vc.changeElement(GCInputButtonB) {
  config in
    let jumpPath = UIBezierPath()
    // load or draw the jump path
    config.path = jumpPath
    return config
}
```


# New controllers and features
* MFI controllers.
* Razor "Kishi", "backbone"

Besides, MFI we support PS,Xbox.  

14.5 added support for new console controllers.  

## DualSense adaptive triggers
* Immersive force feedback
* Programmable resistance and pushback of the trigger
* Emulate a range of real-world sensations

1.  Verify the input profile is a DualSense
2.  Set an effect on `GCAdaptiveTrigger`
3.  Optionally adjust the effect over time

More resistance for rope tension,e tc.

4.  Turn the trigger effect off.

```swift
func updateControllerAdaptiveTriggers() {
  guard let dualSense = GCController.current?.physicalInputProfile as? GCDualSenseGamepad
    else {
        return
    }
  let adaptiveTrigger = dualSense.rightTrigger
  if playerIsPullingSlingshot {
    let resistiveStrength = min(1, 0.4 + adaptiveTrigger.value)
    if adaptiveTrigger.value < 0.9 {
      adaptiveTrigger.setModeFeedbackWithStartPosition(
        0,
        resistiveStrength: resistiveStrength)
    } else {
      adaptiveTrigger.setModeVibrationWithStartPosition(
        0,
        amplitude: resistiveStrength,
        frequency: 0.03)
    }
  } else if adaptiveTrigger.mode != .off {
    adaptiveTrigger.setModeOff()
  }
}
```

## Share buttons

Finally, media capture through GC share buttons.

System gesture is a double-press to capture screenshot, long-press to start/stop replaykit.

instead of having to remember to start/stop recording, players can "turn off automatic background buffering".

RK clips are a great way to capture.

To choose between auto start/stopping, or starting bg recording which you can save 15 ec from, toggle in GC preferences.

API for that.  15 second snapshots etc.

# Wrap up
* Tag your app with game controller support

[[Discover rolling clips with ReplayKit]]
[[What's new in Game Center Widgets, friends, and multiplayer improvements]]




