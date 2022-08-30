# Application design
* more functionality
* more and bigger displays
* keyboard and pointer
* accelerators for common tasks

* navigation bar updates
* find and replace
* edit menu
* collection view improvements

[[Adopt desktop-class editing interactions]]
[[Build a desktop-class iPad app]]

# Styling and controls
* Navigator => default.  Title centered, leading/trailing buttons.  Back button.
* browser => Rearranges to be better optimized for interfaces where history matters.  Files/safari.  Title moved to the leading position.
* editor => Primary function is document editing.  Title is leading, often a destination.  Back button for easy access.

Center items.  UIBarButtonItemGroup, customizationsupport, overflow.  AVailable in all modes, allows the navigator style to intdirectly support center items as well.

Controls continue to be specified as UIBarButtonItems.  Organized as item groups.  Allows for denser presentation when space is at a premium.


```swift
let insertGroup = UIBarButtonItem(title: "Insert", image: UIImage(systemName: "photo"), primaryAction: UIAction { _ in }).creatingFixedGroup()
```

Fixed groups always appear first and cnanot ber removed or moved by customization.

Draw item => movable group.  Cannot be removed, but can be moved.
```swift
// Creating the 'Draw' group

// Convenient form of
// UIBarButtonItemGroup.movableGroup(customizationIdentifier:representativeItem:items:)
let drawGroup = UIBarButtonItem(title: "Draw", …)
    .creatingMovableGroup(customizationIdentifier: "Draw")
```

Optional group => movable and removable.  representative item: collapses the group when necessary.  When customization is invoked, UIKit automatically applies the rules you specify based on how you created your groups.  While fixed and movable groups stay in the bar, optional groups can be added in any number.

```swift
let shapeGroup = UIBarButtonItemGroup.optionalGroup(
    customizationIdentifier: "Shapes",
    representativeItem: UIBarButtonItem(title: "Shapes", image: UIImage(systemName: "square.on.circle")),
    items: [
        UIBarButtonItem(title: "Square", image: UIImage(systemName: "square"), primaryAction: UIAction { _ in }),
        UIBarButtonItem(title: "Circle", image: UIImage(systemName: "circle"), primaryAction: UIAction { _ in }),
        UIBarButtonItem(title: "Rectangle", image: UIImage(systemName: "rectangle"), primaryAction: UIAction { _ in }),
        UIBarButtonItem(title: "Diamond", image: UIImage(systemName: "diamond"), primaryAction: UIAction { _ in }),
    ])
```


Overflow menu contains any items that are part of the customization but oculd not be put into the bar.  As well as customizing the bar.  We synthesize these items by default, you can customize.

```swift
navigationItem.customizationIdentifier = "com.jetpack.blueprints.maineditor"
navigationItem.centerItemGroups = [
    // groups in the default customization
    UIBarButtonItem(title: "Insert", image: UIImage(systemName: "photo"), primaryAction: UIAction { _ in }).creatingFixedGroup(),
    UIBarButtonItem(title: "Draw", image: UIImage(systemName: "scribble"), primaryAction: UIAction { _ in }).creatingMovableGroup(customizationIdentifier: "Draw"),
    .optionalGroup(customizationIdentifier: "Shapes",
                   representativeItem: UIBarButtonItem(title: "Shapes", image: UIImage(systemName: "square.on.circle")),
                   items: [
                    UIBarButtonItem(title: "Square", image: UIImage(systemName: "square"), primaryAction: UIAction { _ in }),
                    UIBarButtonItem(title: "Circle", image: UIImage(systemName: "circle"), primaryAction: UIAction { _ in }),
                    UIBarButtonItem(title: "Rectangle", image: UIImage(systemName: "rectangle"), primaryAction: UIAction { _ in }),
                    UIBarButtonItem(title: "Diamond", image: UIImage(systemName: "diamond"), primaryAction: UIAction { _ in }),
                   ]),
    .optionalGroup(customizationIdentifier: "Text",
                   items: [
                    UIBarButtonItem(title: "Label", image: UIImage(systemName: "character.textbox"), primaryAction: UIAction { _ in }),
                    UIBarButtonItem(title: "Text", image: UIImage(systemName: "text.bubble"), primaryAction: UIAction { _ in }),
                   ]),
    
    // additional group not in the default customization
    .optionalGroup(customizationIdentifier: "Format",
                   isInDefaultCustomization: false,
                   representativeItem: UIBarButtonItem(title: "BIU", image: UIImage(systemName: "bold.italic.underline")),
                   items:[
                    UIBarButtonItem(title: "Bold", image: UIImage(systemName: "bold"), primaryAction: UIAction { _ in }),
                    UIBarButtonItem(title: "Italic", image: UIImage(systemName: "italic"), primaryAction: UIAction { _ in }),
                    UIBarButtonItem(title: "Underline", image: UIImage(systemName: "underline"), primaryAction: UIAction { _ in }),
                   ])
]
```

Enable customization by setting `customizationIdentifier`.  Defines the unique customization of the bar.  Pick a string that won't conflict with other customizations in your app.  UIKit saves/restores customizations based on this identifier.

Format group => optional group.  Here we override the default value iof `isInDefaultCustomization` to exlcude.  

macOS catalyst transltaes to NSToolbar.  Leading, center, trailing added in order.  Customization properties of center item group are respected.  All the expected NSToolbar behaviors are available.


# Document interactions
Adding a menu to the title view, giving a central location for actions that operate on the content as a whole.  Additionally ou can add support for the sahre sheet and drag and drop from this menu.

default title menu items
* duplicate
* move
* rename
* export
* print
```swift
navigationItem.titleMenuProvider = { suggestedActions in
    var children = suggestedActions
    children += [
        UIAction(title: "Comments", image: UIImage(systemName: "text.bubble")) { _ in }
    ]
    return UIMenu(children: children)
}
```

We add a single, additional action to the menu.  You return the composed name.

Allows sharing via activity VC and support for drag and drop.  To enable, provide a UIDocumentProperties instance that describes your document.

```swift
let url = <#T##URL#>
let documentProperties = UIDocumentProperties(url: url)

if let itemProvider = NSItemProvider(contentsOf: url) {
    documentProperties.dragItemsProvider = { _ in
        [UIDragItem(itemProvider: itemProvider)]
    }

    documentProperties.activityViewControllerProvider = {
        UIActivityViewController(activityItems: [itemProvider], applicationActivities: nil)
    }
}

navigationItem.documentProperties = documentProperties
```

Title menui on mac catalyst
suggest items
* exist in file menu
* add additional items via UIMenuBuilder
Document properties
* produces macOS proxy icon

## Rename
Inline rename => UINavigationItem.renameDelegate.  We provide a dedicated UI for editing on all platforms.  Resulting name is passed to the delegate.
"Bring your own" rename => implement UIResponder.rename and providing whatever UI you prefer.

```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        navigationItem.renameDelegate = self
    }
}

extension ViewController: UINavigationItemRenameDelegate {
    func navigationItem(_ navigationItem: UINavigationItem, didEndRenamingWith title: String) {
        // Try renaming our document, the completion handler will have the updated URL or return an error.
        documentBrowserViewController.renameDocument(at: <#T##URL#>, proposedName: title, completionHandler: <#T##(URL?, Error?) -> Void#>)
    }
}
```

# Search
Easier to provide an excellent search experience.  Now takes up less space by being inline in navigation bar on iPadOS.

Restore the old behavior with UINavigationItem.preferredSearchBarPlacement.  Can collapse to a button.  When search is activated, suggestions appear, and they can be updated alongside the updated search query.

Conform to `UISearchResultsUpdating`.


```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        searchController.searchResultsUpdater = self
    }
}

extension ViewController: UISearchResultsUpdating {
    func fetchQuerySuggestions(for searchController: UISearchController) -> [(String, UIImage?)] {
        let queryText = searchController.searchBar.text
        // Here you would decide how to transform the queryText into search results. This example just returns something fixed.
        return [("Sample Suggestion", UIImage(systemName: "rectangle.and.text.magnifyingglass"))]
    }
    
    func updateSearch(_ searchController: UISearchController, query: String) {
        // This method is used to update the search UI from our query text change
        // You should also update internal state related to when the query changes, as you might for when the user changes the query by typing.
        searchController.searchBar.text = query
    }
    
    func updateSearchResults(for searchController: UISearchController) {
        let querySuggestions = self.fetchQuerySuggestions(for: searchController)
        searchController.searchSuggestions = querySuggestions.map { name, icon in
            UISearchSuggestionItem(localizedSuggestion: name, localizedDescription: nil, iconImage: icon)
        }
    }

    func updateSearchResults(for searchController: UISearchController, selecting searchSuggestion: UISearchSuggestion) {
        if let suggestion = searchSuggestion.localizedSuggestion {
            updateSearch(searchController, query: suggestion)
        }
    }
}
```

# Next steps
* adopt center items
* Improve document support
* Add search suggestions
* Get a better mac experience





* https://developer.apple.com/documentation/uikit/uinavigationitem/3967523-titlemenuprovider
* https://developer.apple.com/documentation/uikit/uinavigationitem/itemstyle
* https://developer.apple.com/documentation/uikit/uisearchcontroller/3584821-searchsuggestions
* https://developer.apple.com/documentation/uikit/uinavigationitem/3987967-centeritemgroups
* https://developer.apple.com/documentation/uikit/uinavigationitemrenamedelegate
* https://developer.apple.com/documentation/uikit/uidocumentproperties
* https://developer.apple.com/documentation/uikit/app_and_environment/building_a_desktop-class_ipad_app
* https://developer.apple.com/documentation/uikit/app_and_environment/supporting_desktop-class_features_in_your_ipad_app
