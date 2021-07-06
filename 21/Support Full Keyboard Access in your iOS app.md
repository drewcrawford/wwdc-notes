# Motor accessibility
Accessibility in terms of voiceover, for those who are blind or low vision.  Make sure our software works for anyone with motor impairments.
* Limited range of motion
* Difficulty touching screen
* Alternative inputs


# Full keyboard access
While iOS offers support for keyboards since iOS 9, it's a supplementary mode.

Now can do 100% via keyboard.
Between Assistivetouch and Switch Control for those who may not have enough dexterity to touch the screen, but don't need/want external switches
Alternative to VoiceControl e.g. nonverbal, in an environment where it doesn't maek sense, etc.

Custom or accessible keyboard layouts.  

* Keyboard as input
* Navigation commands
* Interaction commands
* Gestures
* Fully customizable

## Demo
Arrow keys to mvoe, spacebar to launch app
Tab to new note
Immediately edit.  

tab-z to bring up a list of actions.  Space to select item

Tab-f to find.  






# APIs and principles
```swift
// Accessibility custom actions

let addAction = UIAccessibilityCustomAction(
    name: gameLocString("add"), image: UIImage(systemName: "plus.square")) { _ in
            self.addCard()
            return true
        }
        
let pinAction = UIAccessibilityCustomAction(
    name: gameLocString("pin"), image: UIImage(systemName: "pin.fill")) { _ in
            self.pinCard()
            return true
        }
        
cardView.accessibilityCustomActions = [addAction, pinAction]
```

Custom actions will show up as voiceover, switch control, custom access.  Tab-Z.  

## Keyboard shortcuts

* Work for all keyboard users
* Great for accessibility and power users alike
* Hold down command key to see a list

[[Take your iPad apps to the next level]]

```swift
// Keyboard shortcuts

// In AppDelegate.swift
override func buildMenu(with builder: UIMenuBuilder) {
    super.buildMenu(with: builder)
    guard builder.system == .main else {
        return
    }
        
    let pinCommand = UIKeyCommand(title: gameLocString("pin"), image: UIImage(systemName:
        "pin.fill"), action:#selector(GameViewController.pinFocusedCard), input: "P",
        discoverabilityTitle:gameLocString("pin.card"))      
    let addCommand = UIKeyCommand(title: gameLocString("add"), image: UIImage(systemName: 
        "plus.square"), action: #selector(GameViewController.addFocusedCard), input: "A",
        discoverabilityTitle: gameLocString("add.card"))
    let identifier = UIMenu.Identifier("gameplay_menu")
    let menu = UIMenu.init(title: gameLocString("gameplay"), image:  UIImage(systemName
        "rectangle.grid.3x2"), identifier: identifier, children: [addCommand, pinCommand]);
        
    builder.insertSibling(menu, afterMenu: .view)
}
```
Only show these items when a card is selected.

When I hold down command, I see these only?
```swift
// Keyboard shortcuts

// In GameViewController.swift
override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
    if action == #selector(addFocusedCard) || action == #selector(pinFocusedCard) {
        return self.focusedCard != .none
    }
    return super.canPerformAction(action, withSender: sender)
}
```

* Add custom actions and keyboard shortcuts to improve efficiency

Use shift-tab to go back toward the home button.  

Why is our cursor going to an element that we can't interact with?

```swift
itemView.isAccessibilityElement = true
itemView.accessibilityLabel = gameLocString(for: item)
```

VO will also go to "many items" marked with this.

To tell FKA to skip this

```swift
itemView.accessibilityRespondsToUserInteraction = false
```

* Motor technologies show the cursor on rfewer items
* Cursor should only go to interactive items

This is only meaningful if
* `isAccessiblityElement` is true.  
* Element is not interactive

**the cursor should only go to interactive items**

## Focus-based navigation
* Don't override `canBecomeFocused`
* Use accessibility API to modify focus behavior for full keyboard access only
* The focus engine backs Full Keyboard Access

[[Focus on iPad Keyboard Navigation]]
[[Direct and reflect focus in SwiftUI]]
[[Focus interaction in tvOS - 17]]

I want to be sure that I can use FKA's find feature.  

## Supporting accessible input
* Exhaustive list of words people can use to find an element on screen

```swift
self.accessibilityUserInputLabels = [
    gameLocString("settings"),
    gameLocString("prefs"),
    gameLocString("preferences"),
    gameLocString("gear")];
```

Will also help VoiceControl, since they can speak any of these names.

Won't interfere with VO label.

## Apply user input labels to all image-based controls

## Accessiblity paths

```swift
// Accessibility path

let rect = circleLevelButton.convert(levelButton.bounds, to: nil)

circleLevelButton.accessibilityPath = UIBezierPath(ovalIn: rect)


// If your button is in a scroll view, it’s generally better to
// override accessibilityPath and/or accessibilityFrame
extension CircleButton {
    open override var accessibilityPath: UIBezierPath? {
        get {
            let rect = self.convert(self.bounds, to: nil)
            return UIBezierPath(ovalIn: rect)
        }
        set {
            // no-op
        }
    }
}
```

# Key principles
* Add custom actions and keyboard shortcuts to improve efficiency
* The cursor should only go to interactive items
* Apply user input labels to all image-based controls

https://developer.apple.com/accessibility/


