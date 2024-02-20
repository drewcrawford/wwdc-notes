Learn how to bring your UIKit app to visionOS. We'll show you how to build for a new destination, explore APIs and best practices for spatial computing, and take your content into the third dimension when you use SwiftUI with UIKit in visionOS.

For SwiftuI, see

[[Meet SwiftuI for spatial computing]]
[[Elevate your windowed app for spatial computing]]

# Getting started

General tab, and add the new run destination.

New device simulator as the target, and build.  

Some APIs for iPad app are not available.  This is a brand new platform, with brand new capabilities and characteristics.

# Platform differences

When bringing your app over, common areas to investigate.

1.  Deprecated APIs
2. don't translate well to this platform.

No API deprecated prior to iOS 14
move off of deprecated API
adopt new API!!

UIDeviceOrientation
UIScreen
UITabBar leading and trailing

Please check the documentation for more details.
# Polishing your app

In the simulator, clicking with mouse simulates looking at the point.

Note that our bg changes colors, so we might need to change text colors for contrast.  Semantic colors aren't new, but they're especially valuable for this platform.  The apearance of your app should adopt to platform appearance and accessibility settings.  UIColor.label now has many new values.

Semantic colors adapt to platform.  Instead of defining colors with RGB values, instead use a system-provided color that will result in the correct appearance no matter what.  Ex, systemCYAN is a different blue on each paltform.  Also adapt between light/dark.  On this platform, we're vibrant by default on top of glass.

Similarly, using semantic font styles instead of setting static sies, will result in more readable apps.  Also the right thing to do for accessibility.

Can also set textfield's border srtyle to roundedRect, this adds the rescessed appearance.

Materials are a huge cornerstone of this platform. Make your app look beautiful, part of surroundings, etc.  Ensure legibility.  materials adjust to contrast/color balance based on lighting conditions, and colors of objects behind you.

Nod istinction between dark/light appearances.  All builtin controls and containers use vibrant materials by default, ensuring your app looks amazing.

Can override preefrredContainerBackgroundStyle to return automatic, glass, or ?

highlight - helps to make the app feel repsonsive.  Using system components ensures that you get hover effects by default.

Indicate interactivity.  Adding a hover effect will make it easier to target.  One improtant thing about this platform is that exactly where someone is looking is never delivered to app's process.

Brand new API in uikit to add, customize, or disable hover effects.

UIView -> UIHoverStyle.  Can be highlight or lift.

UIShape - abstract representation of shape.

To instead use a rounded rect, need to set hoverstyle property and pass in a rounded rectangle shape.

one last thing to look at, and that's input.  Eyes, hands, pointer, AX.

tap, pan
can reach out and touch it.
trackpad - use that to interact with the system
apple's AX technologies are available on device as well.  VO, switch control, etc.

system GR just works, including trackpad.  But it's important to note max of 2 simultaneous inputs, since each hand can only produce 1 distinct touch.
# Outside the bounds

## Presentations
On iPad, the example app uses sheets, alerts, andp opovers.  Let's go over how those behave on the new platform.

Unlike iPad, we won't dismiss due to touches outside the bounds, regardless of VC's modal presentation properties.

alerts - app's icon is placed at top.  Always present from VC that should be pushed back.

On iPad, popovers are constrained to the scene.  But on spatial platform, this doesn't exist - more like macOS.

If you use standard presentations in iPad, you may be breaking out of the bounds.  As long as you haven't hardcoded any platform assumptions.

###  Build a new trait collection instance from scratch - 16:15
```swift
import UIKit

extension EditorViewController {

    @objc func showDocumentPopover(sender: UIBarButtonItem) {
        let controller = DocumentInfoViewController(document: pixelDocument)
        controller.modalPresentationStyle = .popover
        if let presentationController = controller.popoverPresentationController {
            presentationController.barButtonItem = sender
            if traitCollection.userInterfaceIdiom == .reality {
                presentationController.permittedArrowDirections = .any
            } else {
                presentationController.permittedArrowDirections = .right
            }
        }
        present(controller, animated: true, completion: nil)
    }

}
```
## Ornaments
editor looks a little cramped.  But with ornaments, we can take advantage of extra room the spatial platform provides in a way we never could before.

Ornaments 
* scene-relative placement
* used throughout the platform
* key to shared spatial apps

* leading edge
* above - navigation in safari
* freeform - bottom toolbar

with ornaments, these app keep their primary content in the center and push controls to the edge.  Lifted forward, creating depth.  Outside the bounds, etc.

###  ornaments - 19:46
```swift
extension EditorViewController {

    func showEditingControlsOrnament() {
        let ornament = UIHostingOrnament(sceneAlignment: .bottom, contentAlignment: .center) {
            EditingControlsView(model: controlsViewModel)
                .glassBackgroundEffect()
        }

        self.ornaments = [ornament]

        editorView.style = .edgeToEdge
    }

}
```

ornament inside - leading content alignment.  vs leading (toolbar) and trailing (content)

ornaments share their VC's lifecycle.  If a VC is removed from the hierarchy, its ornaments are too.  Ex, sheet presentations will keep ornaments relative to VC during transitions.

Avoid overlapping

editorView.style = `.edgeToEdge`.  By taking advantage of ornaments, use more of the main area for content.  Making an ornament is simple.  Focus your time and effort on what makes your app unique.
## RealityKIt

New SwiftuI view, RealityView, to host RK content.  Enables entities to be parented in a SwiftUI hierarchy.

[[Build spatial experiences with RealityKit]]

Use SwiftUI from UIKit with UIHostingController.  Take advantage of RealityView without needing to rewrite your UIKit app.  For the example app, I went to use realitykit to bring pixels to life.

###  Build a new trait collection instance from scratch - 22:45
```swift
extension EditorViewController {

    func showEntityPreview() {
        let entityView = PixelArtEntityView(model: entityViewModel)
        let controller = UIHostingController(rootView: entityView)
        addChild(controller)
        view.addSubview(controller.view)
        controller.didMove(toParent: self)
        prepareEditorInteractions()
    }

}
```


###  Build a new trait collection instance from scratch - 22:46
```swift
private let titleLabelTextField: UITextField = {
    textField.textColor = UIColor.label
    return textField
}()

private let authorLabel: UILabel = {
    label.textColor = UIColor.secondaryLabel
    return label
}()
```

###  Build a new trait collection instance from scratch - 22:47
```swift
textField.borderStyle = .roundedRect
```

###  Build a new trait collection instance from scratch - 22:48
```swift
class MyViewController: UIViewController {
    override var preferredContainerBackgroundStyle: UIContainerBackgroundStyle {
        return .glass
    }
}
```

###  Build a new trait collection instance from scratch - 22:49
```swift
class CollectionViewCell: UICollectionViewCell {
    init(document: PixelArtDocument) {
        self.hoverStyle = .init(
            effect: .highlight, 
            shape: .roundedRect(cornerRadius: 8.0))
    }
}
```

###  Build a new trait collection instance from scratch - 22:50
```swift
func fourFingerSwipe() {
    let gesture = UISwipeGestureRecognizer(
        target: self, 
        action: #selector(self.deleteAll))
    gesture.direction = .left
    if traitCollection.userInterfaceIdiom == .reality {
        gesture.numberOfTouchesRequired = 2
    } else {
        gesture.numberOfTouchesRequired = 4
    }
    self.view.addGestureRecognizer(gesture)
}
```

By using standard UIKit presentations, uptting editor controls in an ornament, etc., example app looks great in this new spatial world.  

[[Principles of spatial design]]

# Next steps
* add the new destination in xcode
* update API usage
* Use semantic styles and hover effects
* break outside the bounds
* go even further with swiftui

