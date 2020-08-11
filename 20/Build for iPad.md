#ipados 

Mail has multiple columns
Home has a sidebar.

# Multi-column Split View
`UISplitViewController`
New API in iOS 14
```swift
let splitViewController = UISplitViewController(style: .doubleColumn)
```

Style says up-front, how many columns you want.  Primary, secondary.

```swift
splitViewController.setViewController(sidebarViewController, for: .primary)
splitViewController.setViewController(myHomeViewController, for: .secondary)
```

3-columns

```swift
let splitViewController = UISplitViewController(style: .tripleColumn)
```

1.  Primary (mailboxes)
2.  Supplementary (inbox)
3.  Secondary (message)

* We recommend having 1 app for iPhone and iPad
* Use UISplitViewController as your window's root VC.
	* This displays 2-columns in regular (sometimes 3 on iPad)
	* In compact width, you can specify a separate VC that uses a different navigation scheme.

```swift
splitViewController.setViewController(tabBarController, for: .compact)
```

## Double column display modes
* `secondaryOnly`
* `oneBesideSecondary`
* `oneOverSecondary`

If there are 3 columns
* `twoBesideSecondary`
* `twoOverSecondary` (like the one case, but with 2)
* `twoDisplaceSecondary` -> seems to slide the secondary column over the right

Enable main button and gesture with `presentsWithGesture`
Enable second button with `showsSecondaryOnlyButton`

```swift
splitViewController.preferredSplitBehavior = .tile
```
(If you want columns to be sxs)

```swift
splitViewController.preferredSplitBehavior = .displace
```

```swift
splitViewController.preferredSplitBehavior = .overlay
```

hide/show
```swift
splitViewController.hideColumn(.primary)

splitViewController.showColumn(.supplementary)
```

If you know your app always wants 1 layout, use `preferredDisplayMode`.

```swift
splitViewController.preferredDisplayMode = .oneBesideSecondary
```

Because each column, has an auto UINavigationController.  With a UINavigationBar, and optionally a UIToolBar.  So UIKit puts buttons in those automatically.

`UISplitViewController` docs
* new delegate methods
* Control over column widths
* Animate alongside transitions

HIG: split views

# Lists
> "The modern way to show lists is `UICollectionView`".

[[Advances in UICollectionView]]

## collection view setup
```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .sidebar)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
let collectionView = UICollectionView(frame: frame, collectionViewLayout: layout)
```

## data setup

```swift
struct MyItem: Hashable {
    let title: String
    let image: UIImage
}
```

```swift
let cellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, MyItem> 
{ cell, indexPath, item in

    var content = cell.defaultContentConfiguration()

    content.text = item.title
    content.image = item.image

    cell.contentConfiguration = content
}
```

```swift
let dataSource = UICollectionViewDiffableDataSource<Section, MyItem>
   (collectionView: collectionView)
{ collectionView, indexPath, item in
   return collectionView.dequeueConfiguredReusableCell(using: cellRegistration, 
                                                       for: indexPath,
                                                       item: item)
}
```

For the non-sidebar column, want to use a plain appearance

```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .sidebarPlain)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
let collectionView = UICollectionView(frame: frame, collectionViewLayout: layout)
```

* Accessories
* Reorder 
* Swipe actions


# Reducing modality
Touching outside a menu dismisses it.  And you can keep scrolling with that same touch.
* UIKit dismisses popovers and menu automatically
* Watch for user actions, and dismiss transient UI
# Case study: Shortcuts
## 2-column layout
```swift
let splitViewController = UISplitViewController(style: .doubleColumn)

// Primary column

let sidebar = SidebarViewController()
splitViewController.setViewController(sidebar, for: .primary)


// Secondary column

func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
    splitViewController.showDetailViewController(DetailViewController(), sender: self)
}
```
We set a tab bar for the VC for compact width.
This is the column that we see while in compact width.

```swift
let tabBarController = createTabBarController()

splitViewController.setViewController(tabBarController, for: .compact)
```

Note that we are working with 2 separate hierarchies.  One in regular, one in compact.  So while the split VC knows which one to use, we need to make sure to synchronize them as size classes change.

Here's how we do it.  RVC->Detail.  Detail conforms to `Restorable`.  When the size class is about to change, we look at the current Detail VC, and recreate with the restoreable.

## sidebar
Sidebar is just a VC with a few specific styles.  Inside the sidebar is a UICollectionView.

```swift
let layout = UICollectionViewCompositionalLayout(sectionProvider: sectionProvider,
         configuration: UICollectionViewCompositionalLayoutConfiguration())

//allows us to configure each section in the collection

func sectionProvider(_ section: Int, environment: NSCollectionLayoutEnvironment)
-> NSCollectionLayoutSection {
    var configuration = UICollectionLayoutListConfiguration(appearance: .sidebar)

    if (environment.traitCollection.horizontalSizeClass == .compact) {
        configuration.headerMode = .firstItemInSection
    } else {
        configuration.headerMode = .none
    }

    return NSCollectionLayoutSection.list(using: configuration, layoutEnvironment: environment)
}
```

```swift
//cell registration
struct Section: Hashable { … }

struct Item: Hashable { … }


let cellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, Item> { cell, indexPath, item in
    // Configure the cell
}


let dataSource = UICollectionViewDiffableDataSource<Section, Item>(collectionView: collectionView) { collectionView, indexPath, item in
    return collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: item)
}
```

Avoid subclassing cells by using configurations instead

```swift
let cellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, Item> { cell, indexPath, item in

    var content = cell.defaultContentConfiguration()

    content.text = item.title
    content.image = item.image

    cell.contentConfiguration = content
}
```