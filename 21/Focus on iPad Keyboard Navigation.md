#ipados 

Powerful API to support keyboard navigation in your app

# Keyboard navigation
* Tap navigates between significant areas
* Arrow keys navigate within an area
* Return or space select an item
* Interfering key commands no longer work

 Enabled by default (SDK 15)
 * Text fields
 * Text views
 * Sidebars

Opt in:

* Collection views
* table views
* custom views

Don't make controls focusable
Focus on text, input, list, and grids

[[Suppport full Keyboard access in your iOS app]]

# Focus system
* Save as tvOS
* Same basic APIs
* additional features

[[Focus interaction in tvOS - 17]]

# Focusability
Signle source of truth `canBecomeFocused: Bool`.

Override and return `true`.

What is a focus item?  Backbone of focus system 
* Protocol-based
* Supports custom implementations

Any view can be focused itself, but can also contain focusable subviews.

`UIViewController` only conforms to environment.

Used when rendered with other technologies, such as metal.

```swift
class MyViewController: UICollectionViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        self.collectionView.allowsFocus = true
    }
}
```

Make all cells focusable.  Sidebars allow this by default.

```swift
class MyCollectionViewDelegate: NSObject, UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView,
                canFocusItemAt indexPath: IndexPath) -> Bool {
        return true
    }
}
```

control for each cell individually.  Only effect for cells that don't override themselves.

## Debugging
```lldb
po UIFocusDebugger.checkFocusability(for:)
```

# Appearance
Two styles you'll see.

* Default effect (halo).  `UIFocusHaloEffect`.
* System infers shape, or can customize.

```swift
let focusEffect = UIFocusHaloEffect(roundedRect: self.bounds, cornerRadius: self.layer.cornerRadius, curve: .continuous)
self.focusEffect = focusEffect
```
View hierarchy customization.
```swift
let focusEffect = UIFocusHaloEffect(roundedRect: self.bounds, cornerRadius: self.layer.cornerRadius, curve: .continuous)

// make sure the effect is added right above the image view
focusEffect.referenceView = self.imageView

// make sure the effect is added to our scroll view
focusEffect.containerView = self.scrollView

self.focusEffect = focusEffect
```

Only provide if the inferred appearance is wrong.

## Cell highlighting
* Expected for cells ("only when they have fully opaque content, like an image")
* Automatically with configurations
* Check sample app for custom views

[[Modern cell configuration]]

Cell should look "highlighted".  e.g. foreground becomes tint color?

Not available as a UIFocusEffect.  We adopt this appearance automatically.

If you're not using bg/content configuration, sample app shows how to get the correct color in all cases.

```swift
init(frame: CGRect) {
   super.init(frame: frame)
   self.focusEffect = nil //turn off system styling
}

func didUpdateFocus(in context: UIFocusUpdateContext, withAnimationCoordinator coordinator: UIFocusAnimationCoordinator) {
    if context.nextFocusedItem == self {
        // This view is focused. Customize its appearance.
    }
    else if context.previouslyFocusedItem == self {
        // This view was focused.
    }
}
```

Only make changes in here, when the enxt or previous item is "relevant to this environment".

All ancestor environments... receive a call to this.  So, every superview and VC will get this call.  Allows for flexible implementations.


# Sidebars
Selection follows focus.  

## Selection follows focus
`var selectionFollowsFocus: Bool`

Customizable per cell

```swift
func collectionView(_ collectionView: UICollectionView, selectionFollowsFocusForItemAt indexPath: IndexPath) -> Bool {
    return self.action(for: indexPath).type != .showAlert
}
```

Useful when selecting a cell causes a disruptive action, such as pushing a VC.

e.g. in photos, "New action".

* Property matters for the delegate
* Set the property to the overall intent
* Special case cells with the delegate


# Focus groups
UIKit automatically infers focus groups on the hierarchy, but can also declare them explicitly.

## tvOS movement
* Directional focus.  

## iPadOS/Mac
* arrow keys
* tab

Unlike tvOS, arrow keys only move focus within a defined area.  These areas are called "focus groups".

Tab moves through groups.

Group's "primary item" (first item for the group)
Primary item can change (remembered the last one by system)

Tab key connects the primary items of each group ("the tab loop")

## Default focus groups
Some views define their own group by default
* Scroll views
* Text fields
* Text views

Environments in herit their parent environment's group

`self.focusGroupIdentifier = "com.myapp.groups.sidebar"`


## Primary item

* HIghest priority wins
* Defined system priorities

* Ignored (default)
* previouslyFocused
* prioritized ("selected cell")
* currentlyFocused

Priorities cannot be lowered below the system-priority.

Raise the priority of a different item.

```swift
// Customizing an item’s focus group priority

func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = ...
    if self.isCallToActionCell(at: indexPath) {
        // This cell is not as important as a selected cell but should
        // be chosen over the last focused cell in this group.
        cell.focusGroupPriority = .previouslyFocused + 10
    }
    return cell
}
```

Groups are sorted in reading order (leading-trailing, top-bottom).

Put columns into parent group to force column use.

## Sorting
* Built into UIKit components
* Declare `focusGroupIdentifier` on common ancestor for custom containers
* Define visual structure
* Provide consistent experience


```lldb
po UIFocusDebugger.checkFocusGroupTree(for:)
```

## Focus loop debugger

Launch argument `-UIFocusLoopDebuggerEnabled YES`
# Responder chain
## Responder synchronization
* Focus and first responder synchronize
* Focused item is always inside the first responder

When focus changes, UIKit walks up from the new item to find a responser near the new item.  

When responder changes, UIKit walks up from the new item to find a new focus.

* Key events start at focused item
* Key commands on focus items

If a cell responds to a key command, command is delivered to that cell.

## Become firest responder
* Be conscious about when to call becomeFirstResponder
* Forces focus update
* Best to avoid during focus changes

## Responder chain
* Focus input takes precedence over key commands
* Remap your key commands
* Make sure keyboard navigation works

`keyCommand.wantsPriorityOverSystemBehavior = true`

[[Take your iPad apps to the next level]]

Implement all manual press handling methods (if you handle any)
call super consistently

```swift
override func pressesBegan(_ presses: Set<UIPress>, with event: UIPressesEvent?) {
    if (/* check presses of interest */) {
        // handle the press
    }
    else {
        super.pressesBegan(presses, with: event)
    }
}
```

# Next steps
* make collection and table views focusable
* Update your key commands
* Check out the sample app (search, custom selections, focus guides, etc)

https://developer.apple.com/documentation/uikit/uikeycommand/navigating_an_app_s_user_interface_using_a_keyboard
https://developer.apple.com/documentation/uikit/keyboards_and_input/adjust_your_layout_with_keyboard_layout_guide
https://developer.apple.com/documentation/uikit/focus-based_navigation/about_focus_interactions_for_apple_tv
https://developer.apple.com/documentation/uikit/keyboards_and_input/adding_hardware_keyboard_support_to_your_app
https://developer.apple.com/documentation/uikit/uicommand/adding_menus_and_shortcuts_to_the_menu_bar_and_user_interface
https://developer.apple.com/sample-code/wwdc/2017/Implementing-Advanced-Text-Input-Features.zip



