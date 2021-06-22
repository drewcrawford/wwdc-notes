#ipados 

# Multitasking
[[Introducing Multiple Windows on iPad - 19]]

Scene: single instance of app's UI
components: Scene configuration

* Defines role and delegate class
* Optional attributes
	* Name
	* storyboard
	* scene sublcass
* declare in info.plist
* Can be created at runtime

## scene content
NSUSerActivity
Scene requests
state restoration

Managed by delegate
Sets up UI
Handles lifecycle
Performs state restoration

## Scene tracking
UISceneSession
Scenes can be disconnected
Session always available available

Session - representation in system app switcher.  

`UIWindowScene.ActivationRequestOptions`
Presentation style

## Presentation styles
Influences how scene is presented in the workspace
* Prominent => modally, in the current workspace, behind scene is dimmed.  Cancel/close/done.  Staging ground for new scenes.  Can be repositioned using multitasking.  Moved into the app shelf for later
* standard
* automatic

### Prominent presentation style
* Usefu on its own
* Provide close button
* Dedicated to specific content `UISceneActivationConditions`

[[Targeting content with Multiple Windows]]

### Standard style
e.g., side-by-side.  

### Automatic
Tells the system to choose the best style.  



## Window scene activation
Menus, buttons, and bar button items
Defaults to prominent style
Provides title and image

```swift
let <#newSceneAction#> = UIWindowScene.ActivationAction({ _ in

    // Create the user activity that represents the new scene content.
    let userActivity = NSUserActivity(activityType: <#User Activity Type#>)

    // Return the activation configuration.
    return UIWindowScene.ActivationConfiguration(userActivity: userActivity)

})
```

On iPad and mac catalyst, the menu shows "Open in new window".  On iPhone, the item is automatically hidden because multiple scenes are not supported.  Can provide an alternate action.

```swift
// Create an action to use when multiple scenes are not available.
let alternateAction = UIAction(title: <#Alternate Action Title#>,
                               image: <#Alternate Action Image#>,
                             handler: { _ in
    <#Perform Alternate Action#>
})

// Create the scene activation action with the alternate.
let newSceneAction = UIWindowScene.ActivationAction(alternate: alternateAction) { _ in

    // Create the user activity that represents the new scene content.
    let userActivity = NSUserActivity(activityType: <#Scene Activity Type#>)

    // Return the activation configuration.
    return UIWindowScene.ActivationConfiguration(userActivity: userActivity)
}
```

Clear and famliar way to open content in new scenes.  But not the only way.

In notes, pinching out on a cell opens the note in a new scene.  Scene interactively animates from cell to final position.

## Gestures
1.  UICollectionViewDelegate method
2.  UIWindowScene.ActivationInteraction

Requires the prominent style.

### UICollectionView version

```swift
func collectionView(_ collectionView: UICollectionView,
                    sceneActivationConfigurationForItemAt indexPath: IndexPath,
                    point: CGPoint) -> UIWindowScene.ActivationConfiguration? {

    // Get the item's user activity.
    guard let itemActivity = <#User Activity#> else {
        // Return nil if item can’t be opened in a dedicated scene.
        return nil
    }

    // Return the activation configuration.
    return UIWindowScene.ActivationConfiguration(userActivity: itemActivity)
}
```

### non-UICollectionView version

```swift
// Create an activation interaction.
let newSceneInteraction = UIWindowScene.ActivationInteraction { interaction, point in
    // Get the activity for specific point in view.
    guard let userActivity = <#User Activity#> else { return nil }

    // Return an activation configuration.
    return UIWindowScene.ActivationConfiguration(userActivity: userActivity)

} errorHandler: { error in
	//errors can still occur due to configuration issues or a lack of system resources.
    // Present the content in another manner.
    <#Present Content#>
}

// Add interaction to the view.
<#View#>.addInteraction(newSceneInteraction)
```


### Activation configuration

Common across activation methods
Requires `NSUserActivity`
Allows for customizing
* Request options
* Preview

```swift
// Create the activation configuration.
let itemActivity = NSUserActivity(activityType: <#User Activity Type#>)
let configuration = UIWindowScene.ActivationConfiguration(userActivity: itemActivity)

// If the cell has a subview to use as the preview, create the custom preview.
if let cell = collectionView.cellForItem(at: indexPath) as? <#Expected Cell Class#> {
    configuration.preview = UITargetedPreview(view: cell.<#Subview For Preview#>)
}

// Return the activation configuration.
return configuration
```

By providing a custom preview, the transition is more polished.

Cell transitions from the thumbnail, leaving the rest of the clel in place.

### Preview scene
* `QLPreviewSceneActivationConfiguration`
* Requests system preview scene
* No scene delegate
* App must support multiple windows

Providing polished and convenient ways for content is important.  But you need to save the scene state.

### Saving state

```swift
func stateRestorationActivity(for scene: UIScene) -> NSUserActivity? {
    guard let viewController = self.window?.rootViewController as? <#Expected View Controller Class#> else {
        return nil
    }

    let stateActivity = NSUserActivity(activityType: <#State Restoration Activity Type#>)

    stateActivity.addUserInfoEntries(from: [
        // Save content of a text field.
        <#Content Key#>: viewController.<#Text Field#>.text
    ])

    return stateActivity
}
```

### Interaction state

`UITextField` and `UITextView`
Provides `interactionState` property
* Does not represent content

```swift
func stateRestorationActivity(for scene: UIScene) -> NSUserActivity? {
    guard let viewController = self.window?.rootViewController as? <#Expected View Controller Class#> else {
        return nil
    }

    let stateActivity = NSUserActivity(activityType: <#State Restoration Activity Type#>)

    stateActivity.addUserInfoEntries(from: [
        // Save content of a text field.
        <#Content Key#>: viewController.<#Text Field#>.text,

        // Save interaction state of a text field.
        <#Interaction State Key#>: viewController.<#Text Field#>.interactionState
    ])

    return stateActivity
}
```
 
 ### State restoration pitfalls
 * Restoring on scene connection
	 * Storyboard and views not loaded
 * Restoring when moving to foreground
	 * Track if already performed

### Restoring scene state
* `scene(_:restoreInteractionState:)`
* Scene connected
* Storyboard and views loaded
* Before transition to foreground

```swift
func scene(_ scene: UIScene, restoreInteractionState stateRestorationActivity: NSUserActivity) {
    guard let viewController = window?.rootViewController as? <#Expected View Controller Class#>,
          let userInfo = stateRestorationActivity.userInfo
    else { return }

    if let content = userInfo[<#Content Key#>] as? String {
        // Restore the content first.
        viewController.<#Text Field#>.text = content

        // Then, restore the text field’s interaction state.
        if let interactionState = userInfo[<#Interaction State Key#>] {
            viewController.<#Text Field#>.interactionState = interactionState
        }
    }
}
```

Critical that the content be restored before the interaction state.

### Extending state restoration

* Synchronous state restoration can be hard

Extended state restoration, request a short-term extension.
* Short term
* Keeps launch image visible
* Must signal when complete

Not intended to be used for long-running tasks like network access.

```swift
func scene(_ scene: UIScene, restoreInteractionState stateRestorationActivity: NSUserActivity) {
    guard let viewController = window?.rootViewController as? <#Expected View Controller Class#> else { return }

    // Request an extension.
    scene.extendStateRestoration()

    // Fetch content asynchronously.
    <#self.someAsyncFunction#> { result in
        <#Restore Content#>

        // Signal that state has been restored.
        scene.completeStateRestoration()
    }
}
```




# Keyboard shortcuts

Totally new interface for discovering keyboard shortcuts.  Structure each command into categories (menus).  

Menu offers a search feature that cn be brought up from anywhere.  Tap on a shortcut to trigger it.

## Supporting shortcuts
* `UIKeyCommand` represents a single keyboard shortcut
* Triggered by dispatching to the responder chain

[[Support Hardware Keyboards in your App]]

Mac greys out disabled items
iPad hides them

Mac menu has keyless commands

Main menu system
* System category menus
* System command
* Print command
	* UIApplicationSupportsPrintCommand

### `UIMenuBuilder`
API to customize the main menu
Now supported on iPad apps starting with iPadOS 15
Insert into main menu to categorize key commands

Previously, people used `addKeyCommand` or overriding `keyCommands`.  Now, we recommend adding to main menu by overriding `buildMenu`.

```swift
override func buildMenu(with builder: UIMenuBuilder) {
    super.buildMenu(with: builder)
    
    // Ensure the builder is modifying the main menu.
    guard builder.system == .main else { return }
    
    // Use the builder to modify the main menu...
}
```

Apps should check if they're modifying the main menu system.

Suppose you want a submenu for tabs.

```swift
// Create a menu with key commands.
let tabMenu = UIMenu(options: .displayInline, children: [
    UIKeyCommand(title: NSLocalizedString("New Tab", ...),
                 action: #selector(BrowserViewController.newTab(_:)),
                 input: "t",
                 modifierFlags: .command),
    UIKeyCommand(...)
])

// Insert tabMenu into the File menu.
builder.insertChild(tabMenu, atStartOfMenu: .file)
```

Builtin system menu identifier are defined as constants under `UIMenu.identifier`.  Apps can create their own menu categories.  e.g., "Bookmarks"

```swift
// Create a "Bookmarks" menu.
let bookmarksMenu = UIMenu(title: NSLocalizedString("Bookmarks", ...),
                           children: [...])

// Insert the Bookmarks menu into the root menu, after View.
builder.insertSibling(bookmarksMenu, afterMenu: .view)

// Insert another menu into the Bookmarks menu.
let sortBookmarksMenu = UIMenu(...)
builder.insertChild(sortBookmarksMenu, atEndOfMenu: bookmarksMenu.identifier)
```

UIMenuBuilder will enforce that each element has a unique identifier.  Including commands.

e.g. suppose you have "show as list", "show as grid".  With the same action.  In main menu system, these are implicitly identified by their action, so they won't allow this.

One way to distinguish is to give them different `propertyList` values

A better way is to give them a unique action describing what they do.

Builder enforces that keyboard shortcut combinations are unique.

System italic shortcuts uses cmd-I.  This insertion will fail.  There are two solutions.

1.  App can change shortcut to something new
2.  App can tell the builder to remove the textile commands if not needed.

If an insertion includes a duplicate, error will be logged to console.

On iPad, we don't display the submenu hierarchy.  So for iPad, set a descriptive title.  iPadOS preserves discoverabilityTitle over title.

Responders should still *implement* action methods in main menu commands.  UIKit dispatches to responder.

## Responder traversal
* UIKit looks for a responder that can perform the action
* If it finds one, it calls the action method on that responder

[[Support Hardware Keyboards in your App]]
[[Qualities of a great Mac Catalyst app]]

## Responder methods
* `canPerformAction(_:withSender:)`
* `validate(_:)`

```swift
override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
    if action == #selector(closeTab(_:)) {
        return !openTabs.isEmpty
    } else {
        return super.canPerformAction(action, withSender: sender)
    }
}
```

Asks the responder if it can perform the action
Override to add your own custom logic

By default, this returns true if you implement the action method.

Overrides *must call super for unhandled cases*.

## `validate(_:)`
Updates a commd's properties for the app's current state.

```swift
override func validate(_ command: UICommand) {
    if command.action == #selector(toggleBookmark(_:)) {
        if currentTab.isInBookmarks {
            command.title = NSLocalizedString("Add to Bookmarks", ...)
        } else {
            command.title = NSLocalizedString("Remove from Bookmarks", ...)
        }
    } else {
        return super.validate(command)
    }
}
```

This year, *when apps adopt keyboard navigation with focus system, responder traversal will begin with focused item rather than firstResponder*.

## Focus responder
* Responder chain starts at the focused item
* First responder is further up the chain (not the first responder...)
* Automatic routing of command actions

[[Focus on iPad keyboard navigation]]

## Automatic shortcut localization

* Unreachable keys
* App should not localize modifiers, letting the system do the work
* Opt out of localization

When system localizes shortcuts, it mirrors for right-to-left.  e.g. cmd\[ becomes cmd\].  If you don't ant it mirrored, set `allowsAutomaticMirroring` to `false`.


# Pointer enhancements
Adaptive system pointer.  Bridges between touch-based iPad, and precision of mouse/trackpad.


[[Build for the iPadOS pointer]]
[[Design for the iPadOS Pointer]]

## Band selection in collection views

Built into non-list UICollectionViews via existing API

`collectionView(_ shouldBeginMultiple...)`

## otherwise
* Adopt band selection in any UI
* Implement custom selection behaviors

```swift
// Support multi-selection using UIBandSelectionInteraction.

let selectionInteraction = UIBandSelectionInteraction { [weak self] interaction in
    guard let strongSelf = self else { return }
            
    // Handle selection by responding to interaction state.
    if interaction.state == .selecting {
        strongSelf.selectItemsInRect(interaction.selectionRect)
    } 
    else if interaction.state == .ended {
        strongSelf.finalizeSelection()
    }
}

view.addInteraction(selectionInteraction)
```

Built-in band selection supports common shortcuts out of the box.  e.g., shift.  Command.  

`initialModifiedFlags` are keys held at beginning of interaction
Support common shortcuts like shft-drag or cmd-drag
Custom combinations

## Pointer accessories
Combine shapes with the primary pointer.

Distinctions between accessoreis and custom pointer shape

* Secondary to main pointer shape
* Independent units
* Can be combined with any pointer style

These effects

* Lift efect
* highlight effect
* System style `UIPointerStyle.system()`

System automatically animates appearance and disappearance of accessories.  Also seamlessly animate between shapes.

Transition to communicate change in state or behavior.  e.g `+` vs `no`.

Pointer accessories have a `UIPointerShape` and a `UIPointerAccessoryPosition` (offset from midpoint)

Predefined positions, e.g. `top`, `topRight`, etc.

If the predefined positions don't quite fit, can customize individual properties.

Completely custom position

```swift
// Attach two arrow accessories to a lift pointer effect.

func pointerInteraction(_ interaction: UIPointerInteraction, styleFor region: UIPointerRegion) -> UIPointerStyle?
{
    let preview = UITargetedPreview(view: self)
    let style = UIPointerStyle(effect: .lift(preview))

    if #available(iOS 15.0, *) {
        style.accessories = [
            .arrow(.left),
            .arrow(.right)
        ]
    }

    return style
}
```

## Pointer accessories
* Arrows animate out as the view lifts.

If you've tried to implement this where a view with a pointer effect is draggable, you've probably noticed...

...pointer reaches the edge of the region it disengages from the view.  Usually desireable since it prevents the view from sticking to views as it moves around.  However, maybe we want the pointer effect to remain stable and latch to the view.

To better enable this, iPadOS introduces latching axes on `UIPointerRegion`.

Pointer effect follows the pointer along the axis.  Horizontal => drag along X axis but still rubberband along Y axis.  

These new tools can be used to build new experiences.

1.  Select imag using band selection
2.  Dragging indicators
3.  Pointer hovers
4.  Accessories
5.  Latching allows pointer effect to follow the axis-locked resize gesture.

# Next steps
* Adopt oriminent scenes for uninterrupted focused interactions
* Empower users with new keyboard shortcuts menu
* Boost productivity using new pointer features



