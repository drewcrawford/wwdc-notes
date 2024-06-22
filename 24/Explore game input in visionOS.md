Discover how to design and implement great input for your game in visionOS. Learn how system gestures let you provide frictionless ways for players to interact with your games. And explore best practices for supporting custom gestures and game controllers.

### Respond to a tap gesture - 5:16
```swift
// Respond to a tap gesture.
struct ContentView: View {
    var body: some View {
        RealityView { content in
            // For entity targeting to work, entities must have an InputTargetComponent and a CollisionComponent!
        }
        .gesture(TapGesture().targetedToAnyEntity().onEnded { value in
            print("Tapped entity \(value.entity)!")
        })
    }
}
```

### Combine dragging, magnification, and 3D rotation gestures - 7:08
```swift
// Gesture combining dragging, magnification, and 3D rotation all at once.
var manipulationGesture: some Gesture<AffineTransform3D> {
    DragGesture()
    .simultaneously(with: MagnifyGesture())
    .simultaneously(with: RotateGesture3D())
    .map { gesture in
        let (translation, scale, rotation) = gesture.components()
        return AffineTransform3D(
            scale: scale,
            rotation: rotation,
            translation: translation
        )
    }
}
```

### Create and detect a custom circle gesture - 9:33
```swift
// Create and detect a custom circle gesture.

// Get all required joints and check if they are tracked.
let leftHandIndexFingerTip = leftHandAnchor.skeleton.joint(named: .handIndexFingerTip)
// ...

// Get the position of all joints in world coordinates.
let leftHandIndexFingerTipWorldPosition = matrix_multiply(leftHandAnchor.originFromAnchorTransform, leftHandIndexFingerTip.anchorFromJointTransform).columns.3.xyz
// ...

// Circle gesture detection is true when the distance between the index finger tips centers
// and the distance between the thumb tip centers is each less than four centimeters.
let isCircleShapeGesture = indexFingersDistance < 0.04 && thumbsDistance < 0.04

if isCircleShapeGesture {
    // respond to gesture
}
```

### Detect a connected game controller - 14:00
```swift
// Detect connected game controller.

// Add handler for when controller connects.
NotificationCenter.default.addObserver(forName: NSNotification.Name.GCControllerDidConnect, object: nil, queue: nil) { (note) in
    guard let _controller = note.object? as GCController else { return }
    
    // Add controller input change handlers.
    _controller.physicalInputProfile[GCInputButtonA]?.valueChangedHandler = {
        //...
    }
}

// Poll for controller input
if controller.physicalInputProfile[GCInputButtonA]?.pressed {... }
if controller.physicalInputProfile[GCInputButtonB]?.pressed {... }
```

### Tag a RealityView to handle controller input - 14:24
```swift
// Tag a RealityView to handle controller input.
struct ContentView: View {
    var body: some View {
        RealityView { content in
            // Tag your RealityView to respond to controller input events.
        }
        .handlesGameControllerEvents(matching: .gamepad)
    }
}
```
# Resources

* https://developer.apple.com/documentation/SwiftUI/Composing-SwiftUI-Gestures
* https://developer.apple.com/design/human-interface-guidelines/game-controls
* https://developer.apple.com/design/human-interface-guidelines/gestures
