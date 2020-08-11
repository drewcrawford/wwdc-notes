Muscle memory, ergonomics, etc.
By implementing standard shortcuts, novice users get a familiar, consistent interface.

Your app should excel at both touch and hardware kyboard interactions.

# Keyboard shortcuts

## `UIKeyCommand`
* `UIKeyCommand` -> represents a keyboard shortcut.  title, action, `modifierFlags`, input.
* Override `keyCommands` on `UIResponder` and return an array.

Responder chain loosely follows the view hierarchy of your application.
Your applicatoin's first responder is the UIResponder which all keyboard events funnels into.  If it can't handle, it goes up the chain.

UIViews have their designated `nextResponder` as the view's `superview`.
Key commands are collected from the app from each responder, starting with the first responder, and ending up at the top-level `UIApplication` instance.

Users can discover them via HUD.

## playback play/pause (space)

```swift
class PlayerViewController: UIViewController {
	//first
    override var canBecomeFirstResponder: Bool {
        return true 
    }

	//then.  Make sure this VC is first responder when it first appears
    override func viewDidAppear(_ animated: Bool) {
        becomeFirstResponder()
    }
	//lastly
    override var keyCommands: [UIKeyCommand]? {
        return [
            UIKeyCommand(title: NSLocalizedString("PLAY_PAUSE", comment: "…"),
                        action: #selector(playPause),
                         input: " ")
        ]
    }
}
```

## common shortcuts
| shortcut | action           | `UIResponderStandardEditActions` protocol |
|----------|------------------|-------------------------------------------|
| ⌘X       | Cut              | cut(_:)                                   |
| ⌘C       | Copy             | copy(_:)                                  |
| ⌘V       | Paste            | paste(_:)                                 |
| ⌘A       | Select All       | selectAll(_:)                             |
| ⌘B       | Toggle Bold      | toggleBoldface(_:)                        |
| ⌘I       | Toggle Italic    | toggleItalics(_:)                         |
| ⌘U       | Toggle Underline | toggleUnderline(_:)                       |
| ⌘+       | Increase Size    | increaseSize(_:)                          |
| ⌘-       | Decrease Size    | decreaseSize(_:)                          |

To get these, you don't have to do UIKeyCommand, you just implement the `UIResponderStandardEditActions` protocol (?)

```swift
class SongListTableViewController: UITableViewController {
	
	//still required
    override var canBecomeFirstResponder: Bool {
        return true
    }
    //still required
    override func viewDidAppear(_ animated: Bool) {
        becomeFirstResponder()
    }
    
    /* UIResponderStandardEditActions */

    override func selectAll(_ sender: Any?) { … }

    override func copy(_ sender: Any?) { … }

    override func paste(_ sender: Any?) { … }

}
```
## Working with menus on Catalyst

```swift
class UIKeyCommand : UICommand {
    ...
}

override func buildMenu(with builder: UIMenuBuilder) {
    builder.replaceChildren(ofMenu: .file) { children in
        return [ UIKeyCommand() ] + children
    }
}
```
[[taking iPad apps for mac to the next level - 19]]


# Table views and collection views

## extending selection
e.g., holding shift and selecting contiguously
command to extend selection

```swift
optional func tableView(_ tableView: UITableView,
       shouldBeginMultipleSelectionInteractionAt indexPath: IndexPath) -> Bool

optional func tableView(_ tableView: UITableView,
       didBeginMultipleSelectionInteractionAt indexPath: IndexPath)
```

[[Modernizing your UI for iOS 13 - 19]]

# gesture recognizer

Hold the shift key while resizing the shape with your finger, to constrain to aspect ratio.
cmd - select multiple objects, and then move them all at once.

```swift
func recognizedDragGesture(_ panGesture: UIPanGestureRecognizer) {

    if panGesture.modifierFlags.contains(.command) {
        snapToGrid = true
    } else if panGesture.modifierFlags.contains(.shift) {
        constrainAspectRatio = true
    }
    
    ...
}
```

[[Handle trackpad and mouse input]]

# raw keyboard events
e.g, make small adjustments using the arrow keys.  
* key down -> start moving the object
* key up -> stop moving the object

This isn't possible with `UIKeyCommand` which doesn't send individual events.  They're only fired once.

```swift
class UIResponder: NSObject {
    func pressesBegan(_ presses: Set<UIPress>,
                     with event: UIPressesEvent)
    
    func pressesEnded(_ presses: Set<UIPress>,
                     with event: UIPressesEvent)
}
```
longer, for moving an object

```swift
class CanvasViewController: UIViewController {
     
     override func pressesBegan(_ presses: Set<UIPress>, with event: UIPressesEvent?) {
         for press in presses {
             guard let key = press.key else { continue }
             switch key.keyCode {
             case .keyboardUpArrow: startMoveUp()
             case .keyboardDownArrow: startMoveDown()
                 …
             }
     }
     }

     override func pressesEnded(_ presses: Set<UIPress>, with event: UIPressesEvent?) {
         stopMoving()
     }

}
```
Also, combining with modifier flags
Hold shift to select while moving something
Will also send you events for up/down with modifier keys individually

```swift
class CanvasViewController: UIViewController {

    override func pressesBegan(_ presses: Set<UIPress>, with event: UIPressesEvent?) {
        var selectWhileMoving = false
        for press in presses {
            guard let key = press.key else { continue }
            if key.modifierFlags.contains(.shift) {
                selectWhileMoving = true
            }
                
            switch key.keyCode {
            case .keyboardUpArrow: startMoveUp()
                    
            }
        }
    }
}
```

# wrap up
* Implement common keyboard shortcuts
* Go beyond with customized keyboard shortcuts
* Create menu items so your app is at home on macOS
* Leverage new hardware keyboard APIs

