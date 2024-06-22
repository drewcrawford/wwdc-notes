New in iOS 18, iPadOS 18, and visionOS 2, the PencilKit tool picker gains the ability to have completely custom tools, with custom attributes. Learn how to express your custom drawing experience in the tool picker using the same great tool picking experience available across the system. Discover how to access the new features of the Apple Pencil Pro, including roll angle, the squeeze gesture, and haptic feedback.

### Respond to squeeze in UIKit - 10:24
```swift
class MyViewController: UIViewController, UIPencilInteractionDelegate { 
    func pencilInteraction(_ interaction: UIPencilInteraction, didReceiveSqueeze squeeze: UIPencilInteraction.Squeeze) { 
        if UIPencilInteraction.preferredSqueezeAction == .showContextualPalette && squeeze.phase == .ended { 
            let anchorPoint = squeeze.hoverPose?.location ?? myDefaultLocation 
            presentMyContextualPaletteAtPosition(anchorPoint) 
        } 
    } 
}
```

### Respond to squeeze in SwiftUI - 10:46
```swift
@Environment(\.preferredPencilSqueezeAction) var preferredAction 
@State var contextualPalettePresented = false 
@State var contextualPaletteAnchor = MyPaletteAnchor.default 

var body: some View { 
    MyView() 
        .onPencilSqueeze { phase in 
            if preferredAction == .showContextualPalette, case let .ended(value) = phase { 
                if let anchorPoint = value.hoverPose?.anchor { 
                    contextualPaletteAnchor = .point(anchorPoint) 
                } 
                contextualPalettePresented = true 
            } 
        } 
}
```

### Provide canvas feedback in UIKit - 11:50
```swift
class MyViewController: UIViewController { 
    @ViewLoading var feedbackGenerator: UICanvasFeedbackGenerator 

    override func viewDidLoad() { 
        super.viewDidLoad() 
        feedbackGenerator = UICanvasFeedbackGenerator(view: view) 
    } 

    func dragAlignedToGuide(_ sender: MyDragGesture) { 
        feedbackGenerator.alignmentOccurred(at: sender.location(in: view)) 
    } 

    func snappedToShape(_ sender: MyDrawGesture) { 
        feedbackGenerator.pathCompleted(at: sender.location(in: view)) 
    } 
}
```

### Provide canvas feedback in SwiftUI - 12:29
```swift
@State var dragAlignedToGuide = 0 
@State var snappedToShape = 0 

var body: some View { 
    MyView() 
        .sensoryFeedback(.alignment, trigger: dragAlignedToGuide) 
        .sensoryFeedback(.pathComplete, trigger: snappedToShape) 
}
```

# Resources
* https://developer.apple.com/documentation/ApplePencil
* https://developer.apple.com/documentation/Updates/ApplePencil
* https://developer.apple.com/documentation/pencilkit/configuring_the_pencilkit_tool_picker
* https://developer.apple.com/design/human-interface-guidelines/apple-pencil-and-scribble
* https://developer.apple.com/documentation/ApplePencil/playing-haptic-feedback-in-your-app