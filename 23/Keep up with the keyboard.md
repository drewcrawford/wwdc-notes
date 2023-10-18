#keyboard

Each year, the keyboard evolves to support an increasing range of languages, sizes, and features. Discover how you can design your app to keep up with the keyboard, regardless of how it appears on a device. We'll show you how to create frictionless text entry and share important architectural changes to help you understand how the keyboard works within the system.

We now support many languages, sizes, etc.  Keyboard comes on many different devices, etc.  

Last year, we introduced stage manager.  Multiple scenes, displays, etc.  Use of hardware keyboard and mouse is now more appealing than ever.

# Out of process keyboard
* Privacy and security
* Memory management
* System flexibility

Prior to iOS 17, keyboard's views and logic were in your app's process.  But now we moved to its own process, running almost completely outside your app.

First, let's talk about how it works in-process?

1.  Keyboard requested
2. UI initialization
3. bring up animation
4. Idle/app work
5. touch event
6. text insertion

with out-of-process:
1.  Keyboard requested
2. keyboard async initializes its UI
3. app is idle/app side work
4. keyboard brings up, coordinates animations
5. Now everybody awaits
6. touch events
7. text insertions

Usually, these changes are completely transparent and requires no updates from you.

There are some slight differences in timing.  So if you're sensitive to text entry, timing changes, etc., keep this new architecture in mind.
# Design for the keyboard
common usecase: fullscreen app with the keyboard

With stage manager, apps aren't necessarily fullscreen.  Some special care needs to be taken to adjust the app's view correctly.  Keyboard's scene and app's scene no longer line up.

ex, in this scenario, the app's adjustment isn't actually Y, it needs to adjust by the intersection of the keyboard in your app.

May not be a single adjustment.

Hardware keyboard.  When it's attached, the system will present a toolbar on the center of the screen.  This acts as part of the keyboard, so your views should be adjusted out of the way.  Outside of stage manager, we're preserving existing behavior, but the mini toolbar may overlap your views.

update scroll offsets, etc.

We have code examples.

Keyboard layout guide
* layout guide that matches the keyboard
* recommended approach for integrating keyboard with UIKit

[[Your guide to keyboard layout]]
* follows the keyboard when onscreen and docked
* use bottom safe area when offscreen or undocked
* Follows dismiss gesture when it intersects


###  Keyboard layout guide - 6:21
```swift
view.keyboardLayoutGuide.topAnchor.constraint(equalTo: textView.bottomAnchor).isActive = true
```

customizing the guide
* `followsUndockedKeyboard`
	* set to true to follow even when undocked

* `usesBottomSafeArea`
* * set to false to avoid using the bottom safe area when undocked or onscreen
	* when false, isntead tracks the bottom of the view.  This will allow you to extend your background to cover the bottom of the screen, and also adjust to when the keyboard is brought up.

###  usesBottomSafeArea - 7:56
```swift
// Example of using usesBottomSafeArea to create keyboard and text view aligned with safe area

view.keyboardLayoutGuide.usesBottomSafeArea = false

textField.topAnchor.constraint(equalToSystemSpacingBelow: backdrop.topAnchor, multiplier: 1.0).isActive = true

view.keyboardLayoutGuide.topAnchor.constraint(greaterThanOrEqualToSystemSpacingBelow: textField.bottomAnchor, multiplier: 1.0).isActive = true

view.keyboardLayoutGuide.topAnchor.constraint(equalTo: backdrop.bottomAnchor).isActive = true

view.safeAreaLayoutGuide.bottomAnchor.constraint(greaterThanOrEqualTo: textField.bottomAnchor).isActive = true
```

`keyboardDismissPadding`.  used to add padding.  If you've tried to create an input-accessory-like view.  The keyboard dimiss gesture doesn't begin until the touch intersects.
###  Keyboard dismiss padding - 9:40
```swift
var dismissPadding = aboveKeyboardView.bounds.size.height

view.keyboardLayoutGuide.keyboardDismissPadding = dismissPadding
```

padding above the keyboard that should respond to the dismiss gesture.  Now it begins when the touch intersects with your view.

## Notifications
Before SwiftUI and the keyboard layout guide, you needed to listen for notifications.

System isnt' doing the work for you.

This pattern usually focuses on
1.  getting keyboard notoification
2. getting raw height and using it directly
3. recall earlier that this is the intersection of two rectangles
4. When screen coordinate space and app's coordinate space are different, views can be pushed too high.
5. Few changes to be made to fix this

we started giving you a UIScreen.  Here's how to handle it.

###  Handle willShow or hideKeyboard notifications - 12:11
```swift
func handleWillShowOrHideKeyboardNotification(notification: NSNotification) {
    // Retrieve the UIScreen object from the notification (Added iOS 16.1)
    guard let screen = notification.object as? UIScreen else { return }

    // Determine if the notification’s screen corresponds to your view’s screen
    guard(screen.isEqual(view.window?.screen)) else { return }

    // Calculate intersection with keyboard
    let endFrameKey = UIResponder.keyboardFrameEndUserInfoKey

    // Get the ending screen position of the keyboard
    guard let keyboardFrameEnd = userInfo[endFrameKey] as? CGRect else { return }

    let fromCoordinateSpace: UICoordinateSpace = screen.coordinateSpace
    let toCoordinateSpace: UICoordinateSpace = view

    // Convert from the screen coordinate space to your local coordinate space
    let convertedKeyboardFrameEnd = fromCoordinateSpace.convert(keyboardFrameEnd, to: toCoordinateSpace)

    // Calculate offset for view adjustment
    var bottomOffset = view.safeAreaInsets.bottom

    // Get the intersection between the keyboard's frame and the view's bounds
    let viewIntersection = view.bounds.intersection(convertedKeyboardFrameEnd)

    // Check whether the keyboard intersects your view before adjusting your offset.
    if !viewIntersection.isEmpty {
        // Set the offset to the height of the intersection
        bottomOffset = viewIntersection.size.height
    }

    // Use the new offset to adjust your UI
    movingBottomConstraint.constant = bottomOffset

    // Adjust view layouts and animate using information in notification

    ...

}
```

Keep in mind that we have new out-of-process architecture.  If your app was relying in timing.



# New text entry APIs


* english language keyboard will suggest etxt inline
* Generated using contextual information in the text field
[[What's new with text and text interactions]]

By default, it's usually enabled, but disabled in search/password fields.  You can customize by explicitly setting to yes or no.


###  Inline predictions - 14:38
```swift
@MainActor public protocol UITextInputTraits : NSObjectProtocol {
    // Controls whether inline text prediction is enabled or disabled during typing
    @available(iOS, introduced: 17.0)
    optional var inlinePredictionType: UITextInlinePredictionType { get set }
}

public enum UITextInlinePredictionType : Int, @unchecked Sendable {
    case `default` = 0
    case no = 1
    case yes = 2
}

let textView = UITextView(frame: frame)
textView.inlinePredictionType = .yes
```

# Key takeaways
* design your app to adjust to the keyboard
* keyboard architecture changes may result in subtle timing differences
* adopt APIs that improve the text entry experience


# Resources
* https://developer.apple.com/documentation/SwiftUI/Adding-a-search-interface-to-your-app
* https://developer.apple.com/documentation/uikit/keyboards_and_input/adjusting_your_layout_with_keyboard_layout_guide
* https://developer.apple.com/documentation/SwiftUI/FocusState
* https://developer.apple.com/design/human-interface-guidelines/inputs/keyboards
* https://developer.apple.com/documentation/SwiftUI/SafeAreaRegions
