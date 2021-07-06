# Layout guide
* Register for notifications
* Get applicable frames and animation info from notification
* Do math 
* Adjust layout to match

We're not deprecating notifications.
[[The keys to a better input experience - 17]]

Brand new addition to AL collection: `UIKeyboardLayoutGuide`

## UIKeyboardLayoutGuide
* layout guide
* constrain views to it

```swift
view.keyboardLayoutGuide.topAnchor.constraint(equalToSystemSpacingBelow: textView.bottomAnchor, multiplier: 1.0).isActive = true
```

* Use `view.keyboardLayoutGuide`
* Basic case: update `.topanchor`
* Matches animations
* Follows height changes

Bottom of safe area when undocked


# Integrating the keyboard
## Avoiding "avoidance"
> The keyboard is part of your app.

Your layout should reflect this.

## Follow the leader
* `.followsUndockedKeyboard`
* As the keyboard moves, the guide moves
* No more automatic drop-to-bottom
* `UITrackingLayoutGuide` => tracks constraints that need to change
* Specify constraints active when `near` a specific edge
* Specify constraints active when `awayFrom` a specific edge.

```swift
let awayFromTopConstraints = [
    view.keyboardLayoutGuide.topAnchor.constraint(equalTo: editView.bottomAnchor),
]
//these are active when (keyboard is) away from the top, so they are deactivated when it's near
view.keyboardLayoutGuide.setConstraints(awayFromTopConstraints, activeWhenAwayFrom: .top)

let nearTopConstraints = [
    view.safeAreaLayoutGuide.bottomAnchor.constraint(equalTo: editView.bottomAnchor),

]
view.keyboardLayoutGuide.setConstraints(nearTopConstraints, activeWhenNearEdge: .top)
```
Now, adjust imageview to react to the keyboard

```swift
let awayFromSides = [
    view.keyboardLayoutGuide.centerXAnchor.constraint(equalTo: editView.centerXAnchor),
    imageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
]
view.keyboardLayoutGuide.setConstraints(awayFromSides, activeWhenAwayFrom: [.leading, .trailing])


let nearTrailingConstraints = [
    view.keyboardLayoutGuide.trailingAnchor.constraint(equalTo: editView.trailingAnchor),
    imageView.leadingAnchor.constraint(
        equalToSystemSpacingAfter: view.safeAreaLayoutGuide.leadingAnchor, multiplier: 1.0)
]
view.keyboardLayoutGuide.setConstraints(nearTrailingConstraints, activeWhenNearEdge: .trailing)

let nearLeadingConstraints = [
    editView.leadingAnchor.constraint(equalTo: view.keyboardLayoutGuide.leadingAnchor),
    view.safeAreaLayoutGuide.trailingAnchor.constraint(
        equalToSystemSpacingAfter: imageView.trailingAnchor, multiplier: 1.0)
]
view.keyboardLayoutGuide.setConstraints(nearLeadingConstraints, activeWhenNearEdge: .leading)
```

What `near` and `awayfrom` mean?

* Docked: near bottom, away from other edges
* Undocked can be away from all edges, or can get near the topped edge
* Floating keyboard can be near or away from any edges
	* or two adjacent edges at the same time.

ONly applies for `followsUndockedKeyboard = true`
# Types of keyboards
When you follow th eundocked keyboards, you have extra things to thinnk about.
## Floating
* Can be `awayFrom` everything
* Can be very `near` to top
	* Set contraints when `awayFrom` bottom
* Can be moved at any time

## Split and undocked
* Can be `near` top
* Always `awayFrom` leading/trailing
* Undocked keyboard is `awayFrom` bottom

## Text input via camera
Still a keyboard!
* Same as docked keyboards
* Can be full-screen

[[Use the camera for keyboard input in your app]]

## Hardware keyboards
* Shortcuts bar
* width is adaptive
* Always `near` bottom
* If you collapse it, it can be `near` leading or trailing
* **Width anchor of keyboard can be very small!**

## Multitasking behaviors
* Same as dismissed when out of ap window
* When your app is narrow, `awayFrom` leading and trailing
* Guide is sized for app window

# Wrap up
* Update to KeyboardLayoutGuide
* Go further if it makes sense for your app
* Stop fighting the keyboard



https://developer.apple.com/documentation/uikit/keyboards_and_input/adjust_your_layout_with_keyboard_layout_guide
