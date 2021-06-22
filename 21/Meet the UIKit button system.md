#uikit 

# Buttons
Images, text, both
with/without background
colors

UIKit now provides
* plain
* gray
* tinted
* filled

More features
* Dynamic type
* multiline
* accessibility
* easier customization

# Button configuration
Entrypoint to the new button system. 

Sign in button.

```swift
// Create the Sign In button

let signInButton = UIButton(type: .system)
signInButton.configuration = .filled()
signInButton.setTitle("Sign In", for: [])
```

Integrates titles and images.  Easy to update your button style without updating all your code.

Basic button but we're going to take advantage of new features offered by UIButtonConfiguration.

```swift
// Create the Add to Cart button

var config = UIButton.Configuration.tinted()
config.title = "Add to Cart"
config.image = UIImage(systemName: "cart.badge.plus")
config.imagePlacement = .trailing
addToCartButton = UIButton(configuration: config,
                           primaryAction: …)
```

Subtitle feature.

```swift
// Customize image and subtitle with a configurationUpdateHandler

addToCartButton.configurationUpdateHandler = {
    [unowned self] button in

    var config = button.configuration
    config?.image = button.isHighlighted
        ? UIImage(systemName: "cart.fill.badge.plus")
        : UIImage(systemName: "cart.badge.plus")
    config?.subtitle = self.itemQuantityDescription
    button.configuration = config
}
```

How do we arrange for this to be called?

```swift
// Update addToCartButton when itemQuantityDescription changes

private var itemQuantityDescription: String? {
    didSet {
        addToCartButton.setNeedsUpdateConfiguration()
    }
}
```

With UIButtonConfiguration, there's plenty to like.

## Activity indicator
`showsActivityIndicator` = true

Replaces image if necessary.

## Metrics adjustments
`contentInsets`
`imagePadding`
`titlePadding`

While UIKit lays out automatically, you have control over how these align with each other.

## Semantic styling
* `baseBackgroundColor`
* `baseforegroundColor`
* `cornerStyle`
* `buttonSize`

You automatically get normal, pressed, disabled, etc.

## Customization
* `UIBackgroundConfiguration` support
* `Color` and `TextAttribute` manipulations

```swift
// Configure the button background

var config = UIButton.Configuration.filled()
config.buttonSize = .large
config.image = UIImage(systemName: "cart.fill")
config.title = "Checkout"
config.background.backgroundColor = .buttonEmporium

let checkoutButton = UIButton(configuration: config
                              primaryAction: …) 
addToCartButton.configurationUpdateHandler = {
    [unowned self] button in

    var config = button.configuration
    config?.showsActivityIndicator = self.isCartBusy
    button.configuration = config
}
```


# Toggle buttons
Preserve a `selected `state
Already a state of UIControl
Automatically toggles `selected`

Programmatically change as necessary.

Take advantage of UIButtonConfiguration.

Toggle buttons also work in `UIBarButtonItem`.  

```swift
// Toggle button

// UIAction setup
let stockToggleAction = UIAction(title: "In Stock Only") { _ in
    toggleStock()
}

// The button
let button = UIButton(primaryAction: stockToggleAction)

button.changesSelectionAsPrimaryAction = true

// Initial state
button.isSelected = showingOnlyInStock()
```

# Pop-up buttons

* Similar to pull-down buttons
* Enforce a single selected item
* Display selected item

## Pull-down buttons
* Have a menu assigned
* Present it by default
* Button title is fixed

## Pop-up buttons
* `changesSelectionAsPrimaryAction = true`
* Similar to `UISegmentedControl`
* Allow more/nested choices

Phone app - outgoing line.

```swift
// Pop-up button

let colorClosure = { (action: UIAction) in
    updateColor(action.title)
}

let button = UIButton(primaryAction: nil)

button.menu = UIMenu(children: [
    UIAction(title: "Bondi Blue", handler: colorClosure),
    UIAction(title: "Flower Power", state: .on, handler: colorClosure)
])

button.showsMenuAsPrimaryAction = true

button.changesSelectionAsPrimaryAction = true

// Update to the currently set one
updateColor(button.menu?.selectedElements.first?.title)

// Update the selection
(button.menu?.children[selectedColorIndex()] as? UIAction)?.state = .on
```

[[Build interfaces with style]] -> Interface builder

## Mac catalyst
* Mac buttons have standard appearance
* `UIButtonConfiguration` bridges as appropriate
* Mac appearances
	* Pull-down
	* Pop-up
	* toggle

In some cases, extra customization may be more appropriate.  `button.preferredBehavioralStyle = .automatic` vs `.pad`.  Very prominent custom buttons.

[[What's new in Mac catalyst]]

Much of the functionality is built on top of UIMenu

# Menus

* Subtitles
* Improved submenu interaction
* Selection management.  e.g. toggle within a submenu

On iOS and iPadOS these behaviors are independent of visual customizations.  

```swift
// Single selection menu

// The sort menu
let sortMenu = UIMenu(title: "Sort By", options: .singleSelection, children: [
    UIAction(title: "Title", handler: sortClosure),
    UIAction(title: "Date", handler: sortClosure),
    UIAction(title: "Size", handler: sortClosure)
])

// The top menu
let topMenu = UIMenu(children: [
    UIAction(title: "Refresh", handler: refreshClosure),
    UIAction(title: "Account", handler: accountClosure),
    sortMenu
])

let sortSelectionButton = UIBarButtonItem(primaryAction: nil, menu: topMenu)

updateSorting(sortSelectionButton.menu?.selectedElements.first)
```

# Make better buttons
* Build configurations for buttons
* Add toggle and pop-up buttons
* Remove ("or simplify") button subclasses
* Get automatic Mac Catalyst support