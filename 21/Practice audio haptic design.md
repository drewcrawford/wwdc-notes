

#corehaptics

# Audio and haptic design principles
Flashlight button in iOS.  Combines visual animation, sound, and haptic feedback.  Clear, precise, etc.

To help achieve magic and delight in app experience, there are 3 principles.

* Causality
* Harmony
* Uility

## Causality
For feedback to be useful, it must be obvious what caused it.

## Harmony
It feels the way it looks and the way it sounds

Small ball feels and sounds small.  Large etc.

## Utility
Add audio and haptics that provide clear value to your app experience.

Keep it simple and focus on significant events.
# Introduction to Core Haptics
* Lets you design custom haptic and audio feedback to your app
* Engine
* Player
* Pattern
* Event

## Engine
`CHHapticEngine`.
## Player
Used for playback control
## Pattern
collection of events over time
## Event
building blocks

1.  Transient
2.  continuous

Quicklook visualizer can see ahap file.

For more indepth information, see documentation


# HapticRicochet Xcode project
Simulator doesn't work.  Need iPhone 8 or better.
Make sure you feel and hear the collisions (not silent)

## Shield
Design specifications.
* Visual - 500ms, springy and snappy
* Haptic - transformation, significant new state
* Audio - gain in energy, feels robust and protected

```swift
// Initialize shield.
func initializeShieldHaptics() {
    // Create a pattern from the shield asset.
    let pattern = createPatternFromAHAP("ShieldTransient")!
        
    // Create a player from the shield pattern.
    shieldPlayer = try? engine.makePlayer(with: pattern)
}

/ Play shield transformation.
func shield() {
    // …
    // start player for haptics and audio.
    startPlayer(shieldPlayer)
        
    // Play shield animation
    isAnimating = true
    sphereView.layer.add(shieldAnimation, forKey: "Width")
    // …
}
```

long discussion about harmony between audio and haptics

## Rolling texture

Two issues to resolve
* One technical problem
* One design

Loop the asset so it doesn't stop playing.  AdvancedPatternPlayer can do this.

Match dot pattern intensity to the haptic intensity.  Harmony again.

# Wrap up
* Audio and haptic design principles
* Core Haptics fundamentals
* Hapticricochet.app - shield and rolling texture
* See HIG

[[Expanding the sensory experience with Core Haptics - 19]]

* https://developer.apple.com/documentation/corehaptics/delivering_rich_app_experiences_with_haptics
* https://developer.apple.com/documentation/corehaptics
* https://developer.apple.com/design/human-interface-guidelines/ios/user-interaction/haptics/




