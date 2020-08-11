New xbox controllers

By using the game controller framework, these apps automatically support new added controllers.

# New controller support
`GCPhysicalInputProfile`.  "The new extensible input API".

Every controller now has a physical input profile.  So `GCExtendedGamepad` and `GCMicroGamepad` are now subclasses of this new item.

```swift
// Extensible input API example

var attackComboBtn: GCControllerButtonInput?
var mapBtn: GCControllerButtonInput?
var mappedButtons = Set<GCControllerButtonInput>()
var unmappedButtons = Set<GCControllerButtonInput>()

func setupConnectedController(_ controller: GCController) {
    let input = controller.physicalInputProfile
    
    // Set up standard button mapping
    setupBasicControls(input)
    
    // Map a shortcut to the player's special combo attack
    attackComboBtn = input.buttons["Paddle 1"]
    if (attackComboBtn != nil) { mappedButtons.insert(attackComboBtn!) }
    
    // Map a shortcut to the in-game map
    mapBtn = input.buttons[GCInputDualShockTouchpadButton]
    if (mapBtn != nil) { mappedButtons.insert(mapBtn!) }
    
    // Find buttons that havent' been mapped to any actions yet
    unmappedButtons = input.allButtons.filter { !mappedButtons.contains($0) }
}
```

* `GCDualShockGamepad`
* `GCXboxGamepad`

Use extra buttons to augment your games with additional convenience controls.  Avoid requiring these buttons.
# Haptics and rumble
* Provided by Core Haptics.
* Craft custom haptic patterns
* Play using the haptic engine
* Design once, play everywhere

[[Introducing core haptics - 19]]

1. Design haptic content
2. `CHHapticPattern`
3. Get `GCController`
4. Create `CHHapticEngine`
5.  Create `CHHapticPatternPlayer`.
6.  Can layer these patterns with multiple effets.


## Creating and playing haptics
`createEngineWithLocality`.  `.default` provides a haptic experience that your users would expect.  Typically the handles.

```swift
private func createAndStartHapticEngine() {
    // Create and configure a haptic engine for the active controller
    
    guard let controller = activeController else { return }
    
    hapticEngine = controller.haptics?.createEngine(withLocality: .handles)
    
    guard let engine = hapticEngine else {
        print("Active controller does not support handle haptics")
        return
    }
```

```swift
// Play haptics whenever the player is damaged

private func playerWasDamaged(damage: Float) {
    do {
        // Calculate the magnitude of damage as percentage of range [0, maxPossibleDamage]
        let damageMagnitude: Float = ...
        
        // Create a haptic pattern player for the player being hit by an enemy
        let hapticPlayer = try patternPlayerForPlayerDamage(damageMagnitude)
        
        // Start player, "fire and forget".
        try hapticPlayer?.start(atTime: CHHapticTimeImmediate)
    } catch let error {
        print("Haptic Playback Error: \(error)")
    }
}
```
```swift
// Create a haptic pattern that scales to the damage dealt to the player

private func patternPlayerForPlayerDamage(_ damage: Float) throws -> CHHapticPatternPlayer? {
    let continuousEvent = CHHapticEvent(eventType: .hapticContinuous, parameters: [
        CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.5),
        CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.3),
    ], relativeTime: 0, duration: 0.6)
    
	//note that the time of this will vary from controller to controller
	//because it's the minimum time a controller supports
    let firstTransientEvent = CHHapticEvent(eventType: .hapticTransient, parameters: [
        CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.5),
        CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.9 * damage),
    ], relativeTime: 0.2)
    
    let secondTransientEvent = CHHapticEvent(eventType: .hapticTransient, parameters: [
        CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.5),
        CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.9 * damage),
    ], relativeTime: 0.4)
    
    let pattern = try CHHapticPattern(events: [continuousEvent, firstTransientEvent,
                                               secondTransientEvent], parameters: [])
    
    return try engine.makePlayer(with: pattern)
}
```

## Transitioning from other platforms
* In game engines, haptics are tied to the update loop
* Each update, the intensity of each motor is set directly

```swift
// Update the state of the connected controller's haptics every frame

private func update() {
    updateHaptics()
}

private func updateHaptics() {
    // Update the controller's haptics by sending a dynamic intensity parameter each frame
    do {
        // Create dynamic parameter for the intensity.
        let intensityParam = CHHapticDynamicParameter(parameterID: .hapticIntensityControl,
                                                        value: hapticEngineMotorIntensity,
                                                        relativeTime: 0)
        
        // Send parameter to the pattern player.
        try hapticsUpdateLoopPatternPlayer?.sendParameters([intensityParam],
                                 atTime: CHHapticTimeImmediate)
    } catch let error {
        print("Dynamic Parameter Error: \(error)")
    }
}
```
Will need to repeat this for each motor if you'd like to control them independently.

# current controller
How do you determine which input is active?  Remote or controller?
* Use `GCController.current` to track the most recently used controller

Single player games should adapt UI to active controller

```swift
NSNotification.Name.GCControllerDidBecomeCurrent
NSNotification.Name.GCControllerDidStopBeingCurrentNotification
```
# Motion
DS4 has motion support.  Can use the gyroscope for fine-tune camera angles, with the thumbsticks used for coarse angles.

Also has an accelerometer.  e.g. measures translations.

* Provided via `GCMotion` profile.
* DS4 needs manual motion sensor activation (to preserve battery)

```swift
if motion.sensorsRequireManualActivation {
    motion.sensorsActive = true
}
```

DS4 does not separate gravity from user acceleration.  Only report the total accel of the controller.

```swift
if motion.hasGravityAndUserAcceleration {
    handleMotion(gravity: motion.gravity, userAccel: motion.userAcceleration)
} else {
    handleMotion(totalAccel: motion.acceleration)
}
```

# Lights
Can flash a different color when the player walks into lava.  Or use for a health meter.

* Provided via `GCDeviceLight
* To change a light, just set its color

```swift
guard let controller = GCController.current else { return }
controller.light?.color = GCColor.init(red: 1.0, green: 0, blue: 0)
```

# Battery state
* Provided via `GCDevicePattery`
* level 0 to 1.0
* `charging`, `discharging`, `full`, `unknown`
* Can observe via KVO

# Indicating controller support
Indicate support in xcode
* apps get game controller badge in the app store
* enables per-app controller remapping in settings

# Input remapping

settings, such as inverting the y-axis.  in iOS 14, we have a method to remap.

* Global remapping
	* Applies to all applications
* per-app
* Available if game controller support is indicated

To support global remapping, oyu just need to declare game controller support.
For per-app remapping, check "extensive gamepad" box.


# Unique inputs

We added input glyphs with SFSymbols to assist with communicating which buttons to press.  Different weights etc.  Also light/dark, tinted colors.

[[Introducing SF Symbols - 19]]

Look at `sfSymbolsName` from the controller to get the SF symbols name

```swift
let xboxButtonY = xboxController.physicalInputProfile[GCInputButtonY]!
guard let xboxSfSymbolsName = xboxButtonY.sfSymbolsName else { return }
let xboxButtonYGlyph = UIImage(systemName: xboxSfSymbolsName)

let ds4ButtonY = ds4Controller.physicalInputProfile[GCInputButtonY]!
guard let ds4SfSymbolsName = ds4ButtonY.sfSymbolsName else { return }
let ds4ButtonYGlyph = UIImage(systemName: ds4SfSymbolsName)
```

**Glyphs are input-remapping aware.**

[[Bring Keyboard and Mouse Gaming to iPad]]



