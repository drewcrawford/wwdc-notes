#accessibility 
# Vision loss
Broad continuum.  
Colorblindness, motion sensitivity, etc.

Range of settings so that everyone can get the most out of their device.

# Color and shapes
Using color only to create emphasis is a problem.

Buttons use the color system blue, and some use shapes to help them stand out.
Get additional visualizations.

## Button shapes
If you can't fit button shapes in your ordinary design, you should provide an alternate appearance for people with this setting enabled.

Call `buttonShapesEnabled` on UIAccessibility.

```swift
func observeButtonShapesNotification() {
    // Make buttons more visible by using shapes.
    // If your default design does not include button shapes, observe this notification to make visual changes.
    NotificationCenter.default.addObserver(self, selector: #selector(updateButtonShapes), name: UIAccessibility.buttonShapesEnabledStatusDidChangeNotification, object: nil)
}

@objc func updateButtonShapes() {
    if UIAccessibility.buttonShapesEnabled {
        // Use extra visualizations for buttons.
    } else {
        // Use default design for buttons.
    }
}
```

## Differentiate without color
Use symbols to convey meaning

```swift
func observeDifferentiateWithoutColorNotification() {
    // Use symbols or shapes to convey meaning instead of relying on color alone.
    // If your default design does not differentiate without color, observe this notification to make visual changes.
    NotificationCenter.default.addObserver(self, selector: #selector(updateColorAndSymbols), name: NSNotification.Name(UIAccessibility.differentiateWithoutColorDidChangeNotification), object: nil)
}

@objc func updateColorAndSymbols() {
    if UIAccessibility.shouldDifferentiateWithoutColor {
        // Use symbols or shapes to convey meaning.
    } else {
        // Use default design.
    }
}
```

## Increase constrast
If you're not using the setting, you'll need to be aware of the setting and update accordingly.

Generally, colors should get darker in light mode, and lighter in dark mode.  I know that sounds reverse.

Make changes in the attributes inspector.  Can provide laternate versions of symbols.

Accessibility inspector in Xcode has a super handy color contrast calculator.

## Smart invert colors
Darkens bright white UI elements by inverting system UI
Images, video, and app icons should not invert
Specially flag UIView subclasses taht should keep standard colors (photos, videos, app icons)

```swift
extension UIView {
    @available(iOS 11.0, tvOS 11.0)
    var accessibilityIgnoresInvertColors: Bool { get set }
}
```
## Color and shapes
* Take a variety of approaches to create visual emphasis
* Color and shapes are a great opportunity for branding
* Observe and respect preferences if your default design is not accommodating
* 
# Text Readability
## large text
* Design with large text in mind
* Wrap labels instead of truncating text
* Symbols and glyphs should scale with text

```swift
// ZodiacConstellationCell.swift


override func traitCollectionDidChange (_ previousTraitCollection: UITraitCollection?) {

     if (traitCollection.preferredContentSizeCategory       
         < .accessibilityMedium) { // Default font sizes

         stackView.axis = .horizontal
         stackView.alignment = .center

     } else { // Accessibility font sizes

         stackView.axis = .vertical
         stackView.alignment = .leading

     }
}
```

## bold
For people who need this emphasis on all text in the system.
```swift
func observeBoldTextNotification() {
    // Update labels to use bold or heavy font styles.
    // If you aren't using system font styles, observe this notification to make visual changes.
    NotificationCenter.default.addObserver(self, selector: #selector(updateLabelWeight), name: UIAccessibility.boldTextStatusDidChangeNotification, object: nil)
}

@objc func updateLabelWeight() {
    if UIAccessibility.isBoldTextEnabled {
        // Use bold or heavy font weight
    } else {
        // Use font weight that is default to your design.
    }
}
```

# Display preferences
## reduce motion
* Adjust motion in your app when reduce motion is enabled
* supress idel aniamtions
* parallax, etc.
* even slide transitions

```swift
func observeReduceMotionNotification() {
    // Observe this notification to reduce or remove the frequency and intensity of motion effects.
    NotificationCenter.default.addObserver(self, selector: #selector(updateMotionEffects), name: UIAccessibility.reduceMotionStatusDidChangeNotification, object: nil)
}

@objc func updateMotionEffects() {
    if UIAccessibility.isReduceMotionEnabled {
        // Reduce or remove extraneous motion effects.
    } else {
        // Use default motion effects.
    }
}
```

## prefer crossfade, new API
```swift
func observeCrossFadeTransitionsNotification() {
    // Reduce or remove sliding animations for transitioning views.
    // If you aren't using system-provided navigation, observe this notification to make visual changes.
    NotificationCenter.default.addObserver(self, selector: #selector(updateTransitionEffects), name: UIAccessibility.prefersCrossFadeTransitionsStatusDidChange, object: nil)
}

@objc func updateTransitionEffects() {
    if UIAccessibility.prefersCrossFadeTransitions {
        // Replace sliding transitions with cross-fade animations.
    } else {
        // Use default sliding transitions.
    }
}
```
## Reduce transparency
* Blur effect should become completely opaque
* Improves readibility for text against a blurred background
* Use system blur effects whenever possible, they get this behavior for free?

```swift
func observeReduceTransparencyNotification() {
    // Reduce or remove transparency by adjusting these effects to be completely opaque.
    // If you aren't using system-provided visual effects for blurs or vibrancy, observe this notification to make visual changes.
    NotificationCenter.default.addObserver(self, selector: #selector(updateTransparencyEffects), name: UIAccessibility.reduceTransparencyStatusDidChangeNotification, object: nil)
}

@objc func updateTransparencyEffects() {
    if UIAccessibility.isReduceTransparencyEnabled {
        // Make transparency effects opaque.
    } else {
        // Use default transparency.
    }
}
```