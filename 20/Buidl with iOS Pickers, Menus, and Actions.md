#uikit 
# Control appearance updates
## `UISlider` / `UIProgressView`
* Updated thickness
* Consistency across platforms
* Full Mac control on optimized Catalyst apps
	* Some customization API unused, because these controls look macish.
	* [[Optimize the interfaces of your Mac Catalyst app]]

## `UIActivityIndicatorView`
* New, simpler design
* Use color API and modern styles
* Also updates "pull to refresh"

## `UIPickerView`
Updated styling
**It should be noted that menus may be an appropriate replacement in many contexts**
On catalyst apps on #macOS, menus are almost always the best choice.

## `UIPageControl`
New interactions
Unlimited pages
Enhanced customization APIs
* Optional custom indicators
* Multiple styles

```swift
let pageControl = UIPageControl()
pageControl.numberOfPages = 5

pageControl.backgroundStyle = .prominent //will make background area always show instead of only when interacting

pageControl.preferredIndicatorImage =
    UIImage(systemName: "bookmark.fill")

//customize individual pages
pageControl.setIndicatorImage(
    UIImage(systemName: "heart.fill"), forPage: 0)
```

# Color picker
New view controller for picking colors
Present as sheet
Features
* eyedropper
* favorites
* hexadecimal specification

Supports grabbing colors using eyedropper.  On #ipados can grab colors from multiple apps.  

```swift
var color = UIColor.blue
var colorPicker = UIColorPickerViewController()

func pickColor() {
    colorPicker.supportsAlpha = true
    colorPicker.selectedColor = color
    self.present(colorPicker,
        animated: true,
      completion: nil)
}

func colorPickerViewControllerDidSelectColor(_
  viewController: UIColorPickerViewController) {
    color = viewController.selectedColor
}

func colorPickerViewControllerDidFinish(_
  viewController: UIColorPickerViewController) {
    // Do nothing
}
```

# Date picker
We have made some big improvements to versatility and UX.
* compact style on iOS
* Useful with space constraints
* Full modal calendar when selecting date
* Keyboard for selecting time
* Optimally limit to just date or time
* Better optimized for pointer interaction on #ipados 

Compact style is supported in 10.15.4.  Can also be set by typing in values.


* Inline style on iOS
* Great for iPad or primary UI
* Matches modal presentation
* Optionally limit to just date or time

Creating a datepicker and selecting th date hasn't changed at all on iOS 14.

```swift
let datePicker = UIDatePicker()
datePicker.date = Date(timeIntervalSinceReferenceDate:
                       timeInterval)

datePicker.preferredDatePickerStyle = .compact

datePicker.calendar = Calendar(identifier: .japanese)
datePicker.datePickerMode = .date //if time isn't relevant to this context

datePicker.addTarget(self,
             action: #selector(dateSet),
                for: .valueChanged)
```

# Menus
Add menus to `UIButtons` and `UIBarButtonItems`.

Tapping on a button performs the default action.  Long-pressing presents a menu with more options.  Immediately slide to the item and lift to activate.

Assign `.menu` directly to `UIButton` or `UIBarButtonItem`.
```swift
button.menu = UIMenu(title: "", children: [
    UIMenu(title: "", options: .displayInline, children: (1...2).map { UIAction(title: "Static Item \($0)") { action in }}),
    UIDeferredMenuElement({ completion in
        DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
            completion([UIMenu(title: "", options: .displayInline, children: (1...2).map { UIAction(title: "Dynamic Item \($0)") { action in }})])
        }
    }),
])
```

But what if you don't want to wait?

* `UIButton` -> `button.showsMenuAsPrimaryAction = true`
* `UIBarButtonItem` -> Don't provide a primary action

## Menus in UINavigationBar
Easily pop backwards
Chooses best title from
* `.backBarButtonItem.title`
* `.backButtonTitle`
* `.title`

## `UIControl`

Menus are provided by `UIControl`.  
You can subclass `UIControl` and override `UIContextMenuInteractionDelegate`.  

## `UIDeferredMenuElement`
Asynchronously provide menu items.  UIKit has a loading UI while waiting for menu items.  These menu items are cached should the menu be displayed again.  (I'm wondering how generating these items can possibly be so expensive?)



## `UIContextMenuInteraction`
`updateVisibleMenu` -> modify or replace provided menu.  Display adjusts. 
Children are no longer immutable, so you can update the menu passed to your block.

```swift
self.contextMenuInteraction.updateVisibleMenu { currentMenu -> UIMenu in
    currentMenu.children.forEach { element in
        guard let action = element as? UIAction else { return }
        
        action.state = Bool.random() ? .off : .on
        action.attributes = Bool.random() ? [.hidden] : []
    }
    return currentMenu
}
```

`.menuAppearance == .rich` (preview) or `.compact` (menu-only interactions) or `.none`.  

# UIActions
Make sharing event handling code easier.  See [[Modernize your UI for iOS 13 - 19]].  

## `UIBarButtonItem`
has new initializers.  Only specify parameters you need.

```swift
let saveAction = UIAction(title: "") { action in }
let saveMenu = UIMenu(title: "", children: [
    UIAction(title: "Copy", image: UIImage(systemName: "doc.on.doc")) { action in },
    UIAction(title: "Rename", image: UIImage(systemName: "pencil")) { action in },
    UIAction(title: "Duplicate", image: UIImage(systemName: "plus.square.on.square")) { action in },
    UIAction(title: "Move", image: UIImage(systemName: "folder")) { action in },
])
let optionsImage = UIImage(systemName: "ellipsis.circle")
let optionsMenu = UIMenu(title: "", children: [
    UIAction(title: "Info", image: UIImage(systemName: "info.circle")) { action in },
    UIAction(title: "Share", image: UIImage(systemName: "square.and.arrow.up")) { action in },
    UIAction(title: "Collaborate", image: UIImage(systemName: "person.crop.circle.badge.plus")) { action in },
])
let revertAction = UIAction(title: "Revert") { action in }
self.toolbarItems = [
    UIBarButtonItem(systemItem: .save, primaryAction: saveAction, menu: saveMenu),
    .fixedSpace(width:20.0),
    UIBarButtonItem(image: optionsImage, menu: optionsMenu),
    .flexibleSpace(),
    UIBarButtonItem(primaryAction: revertAction),
]
```

All (?) controls can be constructed with `UIAction`.  But `UIButton` and ? have additional behavior

## `UIButton`
`init(type:primaryAction:)`
type defaults to `.system`
Uses `primaryAction.title` and `primaryAction.image`

Action is registered to handle `.primaryActionTriggered` control event

## `UISegmentedControl`
```swift
let control = UISegmentedControl(frame: frame, actions: Colors.allcases.map { color in
	UIAction(title: color.description) { [unowned imageValue] _ in
		imageView.tintColor = color.color()
	}
})
```

No event handler, no target-action, etc.  No default case, no switch, etc.  Compiler cna help us keep configuration and response to user input in-sync.

By using an enum to generate our actions, a new enum case will update things automatically.

`init(frame:actions:)`
Per-segment action handling
