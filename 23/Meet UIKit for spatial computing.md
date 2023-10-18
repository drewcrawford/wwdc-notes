Learn how to bring your UIKit app to visionOS. We'll show you how to build for a new destination, explore APIs and best practices for spatial computing, and take your content into the third dimension when you use SwiftUI with UIKit in visionOS.

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

###  Build a new trait collection instance from scratch - 19:46
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