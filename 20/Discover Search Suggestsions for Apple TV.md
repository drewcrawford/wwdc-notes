Search suggestions allow you to minimize typing.  Basically a quicktype-like bar.
`UISearchController` does stuff for free


We have a simple tab-based app.
tvOS apps work best with a nav controller that contains a UITabBar controller.

```swift
private let appData: AppData

init(appData: AppData) {
    self.appData = appData
    
    super.init(nibName: nil, bundle: nil)
}

required init?(coder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
}
```

use search tabbar item

```swift
// use the system standard search tab bar item
tabBarItem = UITabBarItem(tabBarSystemItem: UITabBarItem.SystemItem.search, tag: 0)
```

create some ivars

```swift
private let searchController: UISearchController
private let searchContainerViewController: UISearchContainerViewController
```

initialize them

```swift
self.searchController = UISearchController(searchResultsController: self.searchResultsController)
self.searchContainerViewController = UISearchContainerViewController(searchController: searchC
```

viewdidload

```swift
override func viewDidLoad() {
    addChild(searchContainerViewController)
    
    searchContainerViewController.view.frame = view.bounds
    view.addSubview(searchContainerViewController.view)
    searchContainerViewController.didMove(toParent: self)
}
```

we have a collection view that displays our results (not shown).  We want to keep their scroll stuff (?) in sync

```swift
// scroll search controller allong with results collection view
searchController.searchControllerObservedScrollView = searchResultsController.collectionView
```

UISearchResultsUpdating

```swift
searchController.searchResultsUpdater = self
```

Implement `updateSearchResults`
```swift
func updateSearchResults(for searchController: UISearchController) {
    if let searchText = searchController.searchBar.text {
        // get search results for 'searchText' from data source
        let (results, _) = appData.searchResults(seachTerm: searchText, includePhotos: true, includeVideos: true)
        
        searchResultsController.items = results
    } else {
        // no search text, show unfiltered results
        searchResultsController.items = appData.allEntries
    }
}
```

I think that displays the results?

Now we create the search view controller as a tab

```swift
let searchViewController =  SearchViewController(appData: appData)
```


# Suggestions
`UISearchSuggestionItem`
Model that represents the search suggestion in `UISearchController`
Supports an image, text, and description for accessibility
Assign to any `UISearchSuggestion` to `UISearchController` to display.
Can also use your own object, as long as it conforms to `UISearchSuggestion` protocol.

example
```swift
let suggestion1 = UISearchSuggestionItem(localizedSuggestion: "Result1", localizedDescription: "Result1", iconImage: nil)
let suggestion2 = UISearchSuggestionItem(localizedSuggestion: "Result2", localizedDescription: "Result2", iconImage: nil)

searchController.searchSuggestions = [suggestion1, suggestion 2]
```

How to make them update dynamically?  Use `UISearchResultsUpdating`.

Implement UISearchSuggestion properties

```swift
var localizedSuggestion: String? {
    return self.name
}

var iconImage: UIImage? {
    return self.isVideo ? UIImage(systemName: "video") : UIImage(systemName: "photo")
}
```

Implement accessibility description
```swift
var localizedDescription: String? {
    if (self.isVideo) {
        return String.localizedStringWithFormat(NSLocalizedString("%@ - Video", comment: ""), self.name)
    }
    return String.localizedStringWithFormat(NSLocalizedString("%@ - Photo", comment: ""), self.name)
}
```

I clicked on a suggestion of type videos, string blue.  But we have non-video images in search results?

Add new UISearchResultsUpdating to handle suggestion being selected

```swift
func updateSearchResults(for searchController: UISearchController, selecting searchSuggestion: UISearchSuggestion) {
    if let searchText = searchController.searchBar.text {
        var includePhotos = true;
        var includeVideos = true;
        
        
    }
	// check if the suggestion is for a photo or video
	if let suggestedEntry = searchSuggestion as? SuggestedEntry {
		includeVideos = suggestedEntry.isVideo
		includePhotos = !includeVideos
	}
	// filter the results by to include photos, videos, or both
	let (results, _) = appData.searchResults(seachTerm: searchText, includePhotos: includePhotos, includeVideos: includeVideos)

	searchResultsController.items = results
}
```

# Suggestions
Keyboard is very adaptable to languges/input methods.  Consider this.

For IR remotes, we display a grid keyboard, with results to the right and keyboard on left.
For Thai, we use a 3-line keyboard layout on top, with results on bottom.
Keep in mind when sizing search results, because not as many may fit on screen.
Try to avoid covering the keyboard with your own UI, even in the borders outside the safe area, which can lead to focus issues.
Use a symbol image to differentiate between types of content.  SFSYmbols!

# wrap up
* New `UISearchController` for tvOS 14
* Use `UISearchSuggestionItem`

