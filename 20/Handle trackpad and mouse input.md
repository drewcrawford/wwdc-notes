# Common updates
* Pointer movement
* Locking the pointers
* Scroll input

## Pointer movement
### `UIHoverGestureRecognizer`
Driven by `UIEvent.EventType.hover`

```swift
let controlsHover = UIHoverGestureRecognizer(target: self, action: #selector(handleHover))

@objc func handleHover(_ recognizer: UIHoverGestureRecognizer) {
    switch recognizer.state {
    case .began:
        // Pointer entered our view - show controls
        self.showsPlaybackControls = true
    case .ended:
        // Pointer exited our view - hide controls
        self.showsPlaybackControls = false
    default:
        break
    }
}
```
New phases for pointer movement
* `regionEntered` -> pointer entered your window
* `regionMoved` -> pointer is within your window, but has not pressed down
* `regionExited` -> pointer left window

Do not always align with the gesture state.  These phases pertain to the window.

Use UIHoverGestureRecognizer to respond to pointer movement or hiding/revealing content.  Don't use it to modify the appearance of the pointer or apply a hover effect.

Use `UIPointerInteraction` to do that.

[[taking iPad apps for mac to the next level - 19]]
[[Build for the iPadOS Pointer]]

## Locking the pointer
Some apps, like games, like to lock the pointer.
* UIViewController sets preferred value
* Observe resolved value with `UIPointerLockState`
* Preference may not be honored

1.  `prefersPointerLocked = true`
2. system checks requirements
3. system sets `pointerLocked = true`
4. `UIPointerLockState.isLocked` is true

What if your game has an error and you present a `UIAlertController`?  We want a pointer in that case.

The default value of `prefersPointerLocked` is false.  So when `UIAlertController` is presented, `pointerLock` is disabled.

```swift
class GameViewController: UIViewController {
    
    var shouldLockPointer: Bool = true
    
    override var prefersPointerLocked: Bool {
        return self.shouldLockPointer
    }
    
    func disablePointerLock() {
        self.shouldLockPointer = false
        self.setNeedsUpdateOfPrefersPointerLocked()
    }
}
```
Notifications about lock status

```swift
if let pointerLockState = self.window.windowScene?.pointerLockState {
    self.observer = notificationCenter.addObserver(forName: UIPointerLockState.didChangeNotification,
                                                   object: pointerLockState,
                                                   queue: OperationQueue.main) { (note) in
        guard let lockState = note.object as? UIPointerLockState else { return }
        gameEngine.performExpensiveOperationWhile(lockState.isLocked)
    }
}
```

What are those requirements?
Different per-platform

### iPadOS #ipados 
Screne is full screen
* Not in splitview or slideover
* no other app in slide over
* Does not require `UIRequiresFullScreen` key

Scene is `.foregroundActive` e.g. not control center, etc.

### mac
* application is frontmost
* window is frontmost

If your app fails to meet these requirements, pointer lock is disabled.  
However, lock is evaluated continously so you don't have to do anything to get poniter lock back.

Note that pointer locking is not available on all scenes.  In that case, `.pointerLockState` returns `nil`.  (???)

[[Bring Keyboard and Mouse Gaming to iPad]]

## handling scroll input
If users can pan, they expect scroll wheel to work as well.  e.g., sliders for flashlight.

`UIPanGestureRecognizer.allowedScrollTypesMask`
* Give it the scroll types to handle
* Enables handling of `UIEvent.EventType.scroll`
* No mask by default

Note that standard `UIPanGesture` have no mask by default.

### pinch and rotate trackpad gestures
* Use `UIPinchGestureRecognizer` and `UIRotationGestureRecognizer`
* Compatibility mode by default
	* Gesture simulating touches

Out of compatibility mode
* `UIEvent.EventType.transform` from input device
* Requires info.plist key

No additional code required
Careful with `numberOfTouches` (returns 0) and `location(ofTouch:in:)` (may throw exception).

# Advanced updates
* Button mask and keyboard modifiers
* Accepting/rejecting events
* Distinguishing touches
* Opting into new behavior

## button masks and keyboard modifiers

### button masks
* `UIEvent.ButtonMask` is set of buttons pressed
* `UIEvent` and `UIGestureRecognizer` as `.buttonMask`
* Use for specific button clicks
* `UIGestureRecognizer.buttonMask` is from last event processed

`UITapGestureRecognizer.buttonMaskRequred`
* require a button mask before firing

`UIEvent.ButtonMask.buttons()`
* Returns mask for mouse button number
* Use with `.buttonMaskRequired` to target "high number buttons" ??

### keyboard modifiers
`UIKeyModifierFlags` is set of modifiers pressed
UIEvent and UIGestureRecognizer as `.modifierFlags`
Use to alter response
`UIGestureRecognizer.modifierFlags` is from last event processed

[[Support Hardware keyboards in your apps]]

```swift
//respond to a 3rd mouse button clock
self.thirdMouseButtonTap.buttonMaskRequired = .button(3)
```
Show stuff when a modifier is pressed
```swift
func handleHover(_ recognizer: UIHoverGestureRecognizer) {
        
    // Show chapter controls if alt is pressed
    let showChapterControls = recognizer.modifierFlags.contains(.alternate)
        
    // ...
}
```

## accepting or rejecting events

`UIGestureRecognizerDelegate` -> `gestureRecognizer(shouldReceive event:)`
`UIGestureRecognizer` subclass -> `shouldReceive(event:)`
Called only for events handled.  e.g. `UIPinchGestureRecognizer` won't be called for event type `scroll`

Accept or reject based on event type or properties (`buttonMask`, etc.)
Look at properties on `UIEvent`, not ones on `UIGestureRecognizer`.
Move event code in `gestureRecognizer(shouldReceive touch:)`

```swift
//only handle secondary clicks
class SecondaryClickGesture: UIGestureRecognizer {
    
    override func shouldReceive(_ event: UIEvent) -> Bool {
        // Must look at the event’s mask, not the gesture’s 
        return event.buttonMask == .secondary
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent) {
        // Touch handling code ...
    }
}
```

```swift
//only handle secondary clicks or control clicks
class SecondaryClickGesture: UIGestureRecognizer {
    
    override func shouldReceive(_ event: UIEvent) -> Bool {
        // Must look at the event’s properties, not the gesture’s
        let secondaryClick = event.buttonMask == .secondary

        let controlClick = event.buttonMask == .primary && event.modifierFlags == .control 

        return secondaryClick || controlClick
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent) {
        // Touch handling code ...
    }
}
```

```swift
//only receive hover events with the .alternate modifier pressed
let ccHover = UIHoverGestureRecognizer(target: self, 
                                       action: #selector(handleClosedCaptionHover))

ccHover.delegate = self
    
func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, 
                       shouldReceive event: UIEvent) -> Bool {

    if gestureRecognizer == self.closedCaptionHover {
        return event.modifierFlags.contains(.alternate)
    }

    return true
}
```


## Distinguishing touches
`UITouch.TouchType.indirectPointer`
Requires `UIApplicationSupportsIndirectInputEvents`
Use with `UIGestureRecognizer.allowedTouchTypes`


### UIApplicationSupportsIndirectInputEvents
* Not required for general indirect input support
* Required for new touch type and transform event
* Existing projects must opt in
* New projects opted in
* Default in future releases

#### Not present
Compatibility mode
* Clicks are `UITouch.TouchType.direct`
* Pinch and rotate use gesture simulating touches

#### present and true
New features enabled
* Clicks are `UITouch.TouchType.indirectPointer`
* Pinch and rotate use `UIEvent.EventType.transform`

## indirect input warnings
* No touches in scroll or transform event types
* Careful with touch related API
	* `numberOftouches` (returns 0) and `location(ofTouch:in:)` (may throw exception)
	* `gestureRecognizer(shouldReceive touch:)` (code will not be run when driven by these events)
* Incidentally activated gestures will no trigger `UIPinchGestureRecognizer` and `UIRotationGestureRecognizer` are no longer in a compatibility mode

* Gestures respond to multiple event types
* Detect events with `gestureRecognizer(shouldReceive event:)`
* Touch related API with `UIEvent.EventType.touches`
* 