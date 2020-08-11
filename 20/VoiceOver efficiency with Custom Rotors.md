#accessibility 

You use VO by touching the screen to see what's under your finger.
People who aren't looking at the screen rely on rotor.  By twisting 2 fingers like rotating a dial, rotor.

Swipe down -> next rotor
up -> prev rotor

By adding custom rotors, you can transform how a user experiences your app.

* navigate complex interfaces
* find related elements

# Demo
Demo involves a map where users swipe through place markery things from left-to-right.

Which items draw attention visually?
Then maybe group items by category?

We can sort items by distance from the user.  This way, someone can interact with the UI and quickly scan closest items by category.

In both examples, as the user moves through locations, they can quickly determine which locations they're closest to.  Custom rotors make it possible to deliver a similar experience for all users.

# Code
```swift
func customRotor(for poiType: POI) -> UIAccessibilityCustomRotor {
    UIAccessibilityCustomRotor(name: poiType.rotorName) { [unowned self] predicate in
        let currentElement = predicate.currentItem.targetElement as? MKAnnotationView
        let annotations = self.annotationViews(for: poiType)
        let currentIndex = annotations.firstIndex { $0 == currentElement }
        let targetIndex: Int
        switch predicate.searchDirection {
        case .previous:
            targetIndex = (currentIndex ?? 1) - 1
        case .next:
            targetIndex = (currentIndex ?? -1) + 1
        }
        guard 0..<annotations.count ~= targetIndex else { return nil } // Reached boundary
        return UIAccessibilityCustomRotorItemResult(targetElement: annotations[targetIndex],
                                                    targetRange: nil)
    }
}
```
[[Making Apps more Accessible with Custom Actions - 19]]

# Text
Implement a custom rotor to focus on warnings / errors specifically.
```swift
// Custom text rotor

func customRotor(for attribute: NSAttributedString.Key) -> UIAccessibilityCustomRotor {
    UIAccessibilityCustomRotor(name: attribute.rotorName) { [unowned self] predicate in
        var targetRange: UITextRange? // Goal: find the range of following `attribute`
        let beginningRange =
        guard let currentRange =    else { return nil }
        let searchRange: NSRange, searchOptions: NSAttributedString.EnumerationOptions
        switch predicate.searchDirection {   }
        self.attributedText.enumerateAttribute(
            attribute, in: searchRange, options: searchOptions) { value, range, stop in
            guard value != nil else { return }
            targetRange = self.textRange(from: range)
            stop.pointee = true
        }
        return UIAccessibilityCustomRotorItemResult(targetElement: self,
                                                    targetRange: targetRange)
    }
}
```

# Find related elements
* Users can move within categories of elements

# Next steps
* more on text accessibility
* guidance on auditing for accessibility

[[Creating an Accessible Reading Experience - 19]]
[[VoiceOver: App Testing Beyond the Visuals - 18]]

# Wrap up
* Identify the complex visual areas of your interface
* Create custom rotors for related elements

# Resources
https://developer.apple.com/documentation/uikit/accessibility_for_ios_and_tvos
