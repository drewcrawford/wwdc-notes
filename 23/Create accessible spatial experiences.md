#xros  #accessibility 

Learn how you can make spatial computing apps that work well for everyone. Like all Apple platforms, visionOS is designed for accessibility: We'll share how we've reimagined assistive technologies like VoiceOver and Pointer Control and designed features like Dwell Control to help people interact in the way that works best for them. Learn best practices for vision, motor, cognitive, and hearing accessibility and help everyone enjoy immersive experiences for visionOS.

# Overview
While spatial computing are often built with stunning visual features, that doesn't mean that vision or physical movement are required to engage with them.

These experiences have the potential to be incredible impactful to blind/low vision, limited mobility, etc.

Interact with the real world, without having to see what's on the display.  Keep people of all abilities in midn.

Largest list of AX features we've ever included in the first generation of a product.


# Vision
* VoiceOver support
* visual design
* motion

evidently AX shortcut is triple press of digital crown.

* move focus next by pinching right index finger (thumb, index)
* previous : right middle finger
* active: pinch right ring finger, or left index finger

## SwiftUI accessibility
* use standard controls
* adopt accessibility modifiers

[[Accessibility in SwiftUI - 19]]
[[SwiftUI Accessibility Beyond the Basics]]

configure AX properties on realitykit entities
label, value, traits
custom rotors, custom actions, custom content
system actions - activate, adjustable actions, etc.

### Use AccessibilityComponent with RealityKit - 5:28
```swift
var accessibilityComponent = AccessibilityComponent()
accessibilityComponent.isAccessibilityElement = true
accessibilityComponent.traits = [.button, .playsSound]
accessibilityComponent.label = "Cloud"
accessibilityComponent.value = "Grumpy"
cloud.components[AccessibilityComponent.self] = accessibilityComponent

// ...

var isHappy: Bool {
    didSet {
        cloudEntities[id].accessibilityValue = isHappy ? "Happy" : "Grumpy"
    }
}
```

evidently we automatically localize these via type system?

any time you use a convenience property ,the corresopnding property on the AX component is updated appropriately.

VO uses spatial audio to provide cues as to whether objects are located.

* won't receive hand input by default
* allows safe exploration for people using VO

direct gesture mode
* app receives hand input
* VO Does not process its own gestures

People using VO can choose either state based on the app.

to do this, we'll add the activate action to the systemActions property.

### Add an activate action - 8:04
```swift
var accessibilityComponent = AccessibilityComponent()
accessibilityComponent.isAccessibilityElement = true
accessibilityComponent.traits = [.button, .playsSound]
accessibilityComponent.label = "Cloud"
accessibilityComponent.value = "Grumpy"
accessibilityComponent.systemActions = [.activate]
cloud.components[AccessibilityComponent.self] = accessibilityComponent

// ...

content.subscribe(to: AccessibilityEvents.Activate.self, componentType: nil) { activation in
    handleCloudCollision(for: activation.entity, gameModel: gameModel)
}
```

additional APIs
* custom actions
* custom rotors
* custom content

[[Make apps more accessible with Custom Actions - 19]]
[[VoiceOver efficiency with Custom Rotors]]
[[Tailor the VoiceOver experience in your data-rich apps]]

## direct gesture mode.

Post announcements that describe the scene.
### Announce meaningful events and changes in context - 9:23
```swift
AccessibilityNotification.Announcement("8 clouds in front of you").post()
```

announce meaningful events
* heart gesture recognized
* cloud state changes

announce changes in context
* new rooms, environments, or items
use sounds

## visual accessibility
* support dynamic type
* alternate layouts at largest accessibility text sizes
* 4:1 contrast ration between fg and bg color

[[Make your app visually accessible]]

## head anchors
* allow placement of UI relative to a specific point
* head anchors follow your head position
use head anchors sparingly
* difficult for people with low vision
* people might want to get closer to content to read it
* using zoom feature - can't easily position head-anchored content inside the zoom lens, which is also head-anchored

consider using a world anchor
keep head-anchored content decorative
avoid critical information in head-anchored UI
provide alternatives

### Provide alternatives to head anchored content - 13:15
```swift
// SwiftUI
@Environment(\.accessibilityPrefersHeadAnchorAlternative)
private var accessibilityPrefersHeadAnchorAlternative

// UIKit
AXPrefersHeadAnchorAlternative()
NSNotification.Name.AXPrefersHeadAnchorAlternativeDidChange
```

Use alternate anchors.  Observe these and avoid head anchors in these cases.

By default, control center is head-anchored.  As you look around, control center follows you.  While this design makes it easy to access, it might be hcallenging for some people.  Zoom is also head anchored.  This is a feature that magnifies content for people with low-vision.  Control center moves freely about the Y axis.  This lets you position control center inside the zoom lens.


avoid the use of motion that is too rapid
bouncing/wave-like movement
zooming animations
multi-axis movement
spinning/rotating effects
persistent background effects

provide alternatives when reduce motion is enabled

UIAccessibility.isReduceMotionenabled
Utilize crossfades

ex: we make the water static when reduce motion is turned on.
# Motor
Go hands-free with Dwell Control.
Gestures like tap, scroll, long press, and drag.
Design your app to have functionality with this gesture set.

Plan and design your app to support multiple inputs.  Don't accidetnally exclude people!

Pointer control.
* control system focus
* eyes
* head
* wrist
* index finger

Since poitner control can change input signal, use camera-anchored content sparingly.  Prefer world anchors please.

allow multiple avenues for physical interaction.  You never know what kind of disabilities people may have.

not everyone will be able to move in their environment.  Switch control can position you.

**Allow people to bypass experiences that require movement.**
# Cognitive
Guided Access - cognitive feature that promotes focus
restricts people to a single app
minimizes distractions
removes ornamental UI
suppress hardware buttons

[[Create accessible Single App Mode experiences]]

## best practices
* complex gestures can be hard to learn
* create consistent and familiar UI
* give people time to enjoy your app

immersive content can promote focus and attention

not everyone process information at the same speed.  People may need extra time!
# Hearing

Most impactful thing you can do is provide quality captions.

Pop-on captions.  All at once, easy to read.  Insstead of using roll-up captions.

Captiosn can be widely customized.  Font, color, stroke outlines, etc.  These options allow people to customize their captions so theyr'e easy to see and read.

* AVKit
* AVFoundation

Check if closec captions are enabled
`isClosedCaptioningEnabled`
`closedCaptioningStatusDidChangeNotification`

Media accessibility framework
`MACaptionAppearanceCopyFontDescriptorForStyle` and related.

## best practices
* provide high-quality captions
* caption all content
* music
* sound FX
* visually indicate audio source (ex in space)

# Next steps
* set accessibility properties on RealityKit entities
* Provide optiosn for physical interaction
* Remove ambiguity - provide clarity and focus
* care for captioned content for audio experiences










### Provide alternatives when Reduce Motion is enabled - 15:04
```swift
// SwiftUI
@Environment(\.accessibilityReduceMotion)
private var accessibilityReduceMotion

// UIKit
UIAccessibility.isReduceMotionEnabled
UIAccessibility.reduceMotionStatusDidChangeNotification
```

### Check whether captions are enabled - 23:35
```swift
UIAccessibility.isClosedCaptioningEnabled
UIAccessibility.closedCaptioningStatusDidChangeNotification
```
# Resources
* https://developer.apple.com/documentation/visionOS/diorama
* https://developer.apple.com/documentation/visionOS/improving-accessibility-support-in-your-app
* https://developer.apple.com/documentation/uikit/uiaccessibility
* https://developer.apple.com/documentation/swiftui/view-accessibility
* https://developer.apple.com/documentation/mediaaccessibility
