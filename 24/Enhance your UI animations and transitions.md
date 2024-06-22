Explore how to adopt the zoom transition in navigation and presentations to increase the sense of continuity in your app, and learn how to animate UIKit views with SwiftUI animations to make it easier to build animations that feel continuous.

# Transitions

Cell you tap morphs into an incoming views.  Not just a new appearance, it's continuously interactive.  Can drag from the beginning or during a transition.

Zoom transition can increase the sense of continuity in your app by keeping elements onscreen.



###  Zoom transition in SwiftUI - 0:13
```swift
NavigationLink {
    BraceletEditor(bracelet)
        .navigationTransitionStyle(
            .zoom(sourceID: bracelet.id, in: braceletList)
        )
} label: {
    BraceletPreview(bracelet)
}
.matchedTransitionSource(id: bracelet.id, in: braceletList)
```

1.  add `navigationTransitionStyle` modifier.
2. Connect the modifier to a source view, so the system knows where to zoom from.
3. same identifier and namespace in both places!

Or in UIKit,


###  Zoom transition in UIKit - 0:15
```swift
func showEditor(for bracelet: Bracelet) {
    let braceletEditor = BraceletEditor(bracelet)
    braceletEditor.preferredTransition = .zoom { context in
        let editor = context.zoomedViewController as! BraceletEditor
        return cell(for: editor.bracelet)
    }
    navigationController?.pushViewController(braceletEditor, animated: true)
}
```

We need to capture a stable identifier.  Source view can get reused, such as a collectionview.

Note that if the source view can change, grab it later!

UIKit appearance callbacks.  


Normally,
1.  Disappeared state
2. viewWillAppear, isAppearing, didAppear
3. Appeared state

Pop
1.  willDisappear, didDisappear
2. Disappeared state

Cancellation:
1.  viewWillDisappear
2. but not did?
3. then back through appearing things?

More complex
1.  Appeared
2. appearing (will, is?)
3. now we try to cancel, but instead it appears immediately.  Then the pop starts, moving to disappearing state.

Cancelling push and pop are different, on purpose.  Conceptually the system never cancels an interrupted push.  It's converted into a pop.  From the perspective of a pushed VC, we always reach appeared state.

Be ready for new transitions at any time.  Don't handle being in atrasition differently than not.  Just call  push, regardless of whether a transition is running or not.

Minimize transition state.  The less state you have, the less likely you are to make other code dependent on transition state.  Reset it by viewdidAppear, viewdiddisappear.  

Incorporate SwiftUI.  


# SwiftUI animation



###  Animate UIView with SwiftUI animation - 0:54
```swift
UIView.animate(.spring(duration: 0.5)) {
    bead.center = endOfBracelet
}
```

Use SwiftUI animation types to animate UIKit/AppKit views.  

If your code works with CALayers, some things to worry about.
Existing UIKit API adds a CAAnimation.  However, SwiftUI animation does not create CAAnimation, but animates presentation values directly.  Still reflected in the presentation layer.



# Animating representables
###  Animating representables - 1:00
```swift
struct BeadBoxWrapper: UIViewRepresentable {
    @Binding var isOpen: Bool
    
    func updateUIView(_ box: BeadBox, context: Context) {
        context.animate {
            box.lid.center.y = isOpen ? -100 : 100
        }
    }
}

struct BraceletEditor: View {
    @State private var isBeadBoxOpen = false
    
    var body: some View {
        BeadBoxWrapper($isBeadBoxOpen.animated())
            .onTapGesture {
                isBeadBoxOpen.toggle()
            }
    }
}
```


# Gesture-driven animations

Just pass swiftUI animation types to get swiftUI-style velocity matching!

###  Gesture-driven animations - 1:06
```swift
switch gesture.state {
case .changed:
    UIView.animate(.interactiveSpring) {
        bead.center = gesture.translation
    }
case .ended:
    UIView.animate(.spring) {
        bead.center = endOfBracelet
    }
}
```

It's up to you

* adopt zoom transitions
* always be ready to transitions
* Use SwiftUI animations!  Especially in UI where maintaining continuous velocity is important


[[Explore SwiftUI animation]]
[[Animate with springs]]

# Resources
* https://developer.apple.com/documentation/SwiftUI/Unifying-your-app-s-animations
