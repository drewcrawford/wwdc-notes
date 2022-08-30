We'll update an existing iPad app toa desktop-class experinence.

As we walk through each step, I'll discuss best practices and motivations behind our choices.  Giving you an idea of the factors you should consider.

[[Meet desktop-class iPad]]
[[What's new in iPad app design]]

# UI organization
Since the app is designed for iPad OS 15 ( #ipados ) we place ocntrols in menus and popovers.  In 16, we formalize the existing navigation style and add two new ones.  Express the layout most appropriate to the content.

* Navigator
	* Familiar, push/pop navigation model.  Generally appropriate for apps that display hierarhcical data like settings.
* Browsers
	* Ideal for looking through and navigating back and ofrth between multiple documents
* Editors
	* Focused viewing or editing of individual documents

For markdown, we'll pick the Editor style.  Leading title.  Opening up center for a new set of items.  Expose additional functionality that might have been hidden away.
```swift
navigationItem.style = .editor
```
* back action
* Title menu with document info
* Document renaming
* Center items

 Remove trailing done and replace with `backAction` API.
 ```swift
navigationItem.backAction = UIAction(…)
```

## Title menu
* menu attached to title
* actions for entire context
If your ap isn't document based, it may be a good place to seurface actions applying the whole view.
* document info at a glance
* drag and drop
* sharing


```swift
let properties = UIDocumentProperties(url: document.fileURL)

if let itemProvider = NSItemProvider(contentsOf: document.fileURL) {
    properties.dragItemsProvider = { _ in
        [UIDragItem(itemProvider: itemProvider)]
    }
    properties.activityViewControllerProvider = {
        UIActivityViewController(activityItems: [itemProvider], applicationActivities: nil)
    }
}

navigationItem.documentProperties = properties
```

Since we specified drag item and activity view providers, I can drag to copy the document or tap the share button.

Title menu is also a place for menu.  
* system provided actions
* custom actions
## Renaming
* rename delegate
* rename UI built into bar
* action automatically added to menu
```swift
override func viewDidLoad() {
    navigationItem.renameDelegate = self
}

func navigationItem(_ navigationItem: UINavigationItem, didEndRenamingWith title: String) {
    // Rename document using methods appropriate to the app’s data model
}
```

The app's repsonsibility to handle by renaming the document.  API is intentionally open-ended to support any kind of data model you may have.

First we need to override functions on our editor VC.

```swift
override func duplicate(_ sender: Any?) {
    // Duplicate document
}

override func move(_ sender: Any?) {
    // Move document
}

func didOpenDocument() {
    ...
    navigationItem.titleMenuProvider = { [unowned self] suggested in
        var children = suggested

        ...

        return UIMenu(children: children)
    }
}
```

add custom actions, or whole menu categories.

```swift
func didOpenDocument() {
    ...
    navigationItem.titleMenuProvider = { [unowned self] suggested in
        var children = suggested
        children += [
            UIMenu(title: "Export…", 
                   image: UIImage(systemName: "arrow.up.forward.square"), 
                   children: [
                UIAction(title: "HTML", image: UIImage(systemName: "safari")) { ... },
                UIAction(title: "PDF", image: UIImage(systemName: "doc")) { ... }
            ])
        ]
        return UIMenu(children: children)
    }
}
```

Note that the custom actions are *not called on mac catalyst*.  So we need to manually add them via UIMenu!

## Centeritems
Include a large set of controls without worrying about filling the UI with unused ones.  Each person can tailor the contents.

```swift
navigationItem.customizationIdentifier = "editorView"
```

Groups.  

```swift
UIBarButtonItem(title: "Sync Scrolling", ...).creatingFixedGroup()
```

This is an important function.  Let's place it in a fixed group.  Not customizable and cannot be moved by the user.

```swift
UIBarButtonItem(title: "Add Link", ...).creatingOptionalGroup(customizationIdentifier: "addLink")
```

This one is not critical and other ways to achieve functionality.  So let's make it optional.  Customization identifier to persist across launches.


```swift
UIBarButtonItemGroup.optionalGroup(customizationIdentifier: "textFormat",
																	 isInDefaultCustomization: false,
																	 representativeItem: UIBarButtonItem(title: "Format", ...)
																	 items: [
																	      UIBarButtonItem(title: "Bold", ...),
																	      UIBarButtonItem(title: "Italics", ...),
																	      UIBarButtonItem(title: "Underline", ...),
																	 ])
```

This isn't improtant enough to show by default, but we want it available for customization.  So we'll use the initializer that says it's not in there by default.

Also give it a compact representation.

On mac, center items have been converted to fully customizable NSToolbar items.

* Items automatically overflow into menu
* Custom menu representations
```swift
sliderGroup.menuRepresentation = UIMenu(title: "Text Size",
                                        preferredElementSize: .small,
                                        children: [
    UIAction(title: "Decrease",
             image: UIImage(systemName: "minus.magnifyingglass"),
             attributes: .keepsMenuPresented) { ... },
    UIAction(title: "Reset",
             image: UIImage(systemName: "1.magnifyingglass"),
             attributes: .keepsMenuPresented) { ... },
    UIAction(title: "Increase",
             image: UIImage(systemName: "plus.magnifyingglass"),
             attributes: .keepsMenuPresented) { ... },
])
```

# Quick actions
Prior to iOS 16, adding the ability to edit multiple items... was annoying.

iOS 16 introduces a new design for multitem menus.
* new design
* affected items flock together
* transition to drag and drop

Lightweight selection style.
* multiple selection without edit mode
* No change to app's UI

Achieve this ane enable keyboard focus using existing APi...
```swift
// Enable multiple selection
collectionView.allowsMultipleSelection = true

// Enable keyboard focus
collectionView.allowsFocus = true

// Allow keyboard focus to drive selection 
collectionView.selectionFollowsFocus = true
```

Since we allow multiple selection outside of edit mode, this runs on every selection.  Instead, let's implement `performPrimaryActionForItemAtIndexPath`.  This fucntion is only called when a single item is tapped and the collection view is not editing, we don't need to check for edit mode.

Since we don't have any selection-related behavior, we can remove `didSelectItemAt`.
```swift
func collectionView(_ collectionView: UICollectionView, 
                    performPrimaryActionForItemAt indexPath: IndexPath) {
  
    // Scroll to the tapped element
    if let element = dataSource.itemIdentifier(for: indexPath) {
        delegate?.outline(self, didChoose: element)
    }
}
```

Can now display menus for 0-many items.  Number of items depends on how many items are selected.
```swift
func collectionView(_ collectionView: UICollectionView, 
                      contextMenuConfigurationForItemsAt indexPaths: [IndexPath], 
                      point: CGPoint) -> UIContextMenuConfiguration? {

    if indexPaths.count == 0 {
        // Construct an empty space menu (blank space between cells)
    } 
    else if indexPaths.count == 1 {
        // Construct a single item menu (deselected, or sole selected item)
    } 
    else {
        // Construct a multi-item menu (item that was part of a multiple selection)
    }
}
```

# Text experience
* enable in UITextView with a single bool
* `isFindInteractionEnabled = true`
* Adding custom action to UITextview's builtin edit menu

```swift
textView.isFindInteractionEnabled = true
```

Construct and return a UImenu that combines custom actions with the system menu.
```swift
func textView(_ textView: UITextView,
              editMenuForTextIn range: NSRange,
              suggestedActions: [UIMenuElement]) -> UIMenu? {
    
    if textView.selectedRange.length > 0 {
        let customActions = [ UIAction(title: "Hide", ... ) { ... } ]
        return UIMenu(children: customActions + suggestedActions)
    }
    
    return nil
}
```

[[Adopt desktop-class editing interactions]]

Making our app desktop-class and translating it to the mac.  Use these APIs to take your own app through the process

# next steps
* choose a navigation style
* enhance document owrkflows
* Surface functionality in center area
* Enable quick actions
* Adopt editing interactions
* 
* https://developer.apple.com/forums/tags/wwdc2022-10070
* https://developer.apple.com/forums/create/question?&tag1=141&tag2=382030&tag3=155
* https://developer.apple.com/documentation/uikit/uicollectionviewdelegate/3975794-collectionview
* https://developer.apple.com/documentation/uikit/uinavigationitem/3967523-titlemenuprovider
* https://developer.apple.com/documentation/uikit/uinavigationitem/itemstyle
* https://developer.apple.com/documentation/uikit/uicollectionviewdelegate/4002186-collectionview
* https://developer.apple.com/documentation/uikit/uinavigationitem/3987967-centeritemgroups
* https://developer.apple.com/documentation/uikit/uinavigationitemrenamedelegate
* https://developer.apple.com/documentation/uikit/uidocumentproperties
* https://developer.apple.com/documentation/uikit/app_and_environment/building_a_desktop-class_ipad_app
* https://developer.apple.com/documentation/uikit/app_and_environment/supporting_desktop-class_features_in_your_ipad_app
