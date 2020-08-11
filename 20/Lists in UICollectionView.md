#uikit 
[[advances in collectionview]]

# 
UITableView-like appearance
Based on compositional layout
customizable
optimized self-sizing -> new default behavior

Can still override preferredLayoutAttributes fittingAttributes?



# Components of a list
* `UICollectionLayoutListConfiguration` <- List
* `NSCollectionLayoutSection` <- compositional layout
* `UICollectionViewCompositionalLayout` <-compositional layout

[[Advances in collectionview layout - 19]]

# List configuration
Like tableview styles

* appearance
* separators
* headers
* footers

# Creating a list
When possible
```swift
// Simple setup     
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

More-powerful way.  But instead of creating a compositional layout, you create a section.

```swift
// Per section setup
     
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let section = NSCollectionLayoutSection.list(using: configuration, layoutEnvironment: layoutEnvironment)
```

full example, customize each layout per section
```swift
// Per section setup

let layout = UICollectionViewCompositionalLayout() {
    [weak self] sectionIndex, layoutEnvironment in
    guard let self = self else { return nil }

    // @todo: add custom layout sections for various sections
  
    let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    let section = NSCollectionLayoutSection.list(using: configuration, layoutEnvironment: layoutEnvironment)
    return section
}
```

Now that we have this setup, we can customize our layout on a per-section basis.

# Headers/footers
Work differently than UITableView.
Have to be explicitly enabled.  First, we could register as a supplementary view:

```swift
var configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
configuration.headerMode = .supplementary
let layout = UICollectionViewCompositionalLayout.list(using: configuration)

dataSource.supplementaryViewProvider = { (collectionView, elementKind, indexPath) in
    if elementKind == UICollectionView.elementKindSectionHeader {
        return collectionView.dequeueConfiguredReusableSupplementary(using: header, for: indexPath)
    }
    else {
        return nil
    }
}
```
Keep in mind, you have to provide a supplementary view when collectionview asks.  If you return `nil`, collectionview will assert.  So if some sections require a header while others don't, use per-section configuration.

```swift
let layout = UICollectionViewCompositionalLayout() {
    [weak self] sectionIndex, layoutEnvironment in
    guard let self = self else { return nil }

    // check if this section should show a header, e.g. by implementing a shouldShowHeader(for:) method.
    let sectionHasHeader = self.shouldShowHeader(for: sectionIndex)
  
    let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    configuration.headerMode = sectionHasHeader ? .supplementary : .none
    let section = NSCollectionLayoutSection.list(using: configuration, layoutEnvironment: layoutEnvironment)
    return section
}
```

Alternatively, for headers, we can use `.firstItemInSection` instead

```swift
var configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
configuration.headerMode = .firstItemInSection
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

Data source needs to be aware
used for collapsable sections
[[advances in diffable datasources]]

# Presentation styles
iOS 14 introduces new `UICollectionViewListCell`

## List cell
Can use a list cell anywhere the UICollectionView cell is expected.  Just pick the bits and pieces of the API that you need

* Separators
* Indentation
* Swipe actions
* Accessories
* Default content configuration

[[Modern cell configuration]]

### separators

How to inset a separator?  Hard to align with a label if you specify the point size because layout may change due to DT, etc.
In List cell we're introducing a new `separatorLayoutGuide`.  Instead of constraining your content to this layout guide, you constrain this layout guide *to* your content.

Configure your cell's layout first, and then constrain the `separatorLayoutGuide.leadingAnchor = label.leadingAnchor`

Note that with the system layouts we do this for you, you only need this for custom layouts.

### swipe actions

In contrast to UITableView, swipe actions are now a part of list cell.

Configured like cell content
Communicates with list section
Override getter for dynamic lookup.  (where you only create configuration when the swipe is about to start).

```swift
let cellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, Model> { (cell, indexPath, item) in
    // @todo configure the cell's content
    
    let markFavorite = UIContextualAction(style: .normal, title: "Mark as Favorite") {
        [weak self] (_, _, completion) in
        guard let self = self else { return }
        // trigger the action with a reference to the model
        self.markItemAsFavorite(with: item.identifier)
        completion(true)
    }
    cell.leadingSwipeActionsConfiguration = UISwipeActionsConfiguration(actions: [markFavorite])
}
```

We will only call the getter when the user starts to swipe the cell.
*Never capture the index path of the cell you are configuring inside an action.*
You might actually operate on the data of another cell.  This is particularly dangerous for the delete action.  Instead, either capture the data model directly, or a stable identifier.

### accessories

List Cell offers many new accessory types, and leading/trailing, and multiple accessories per side.

If you configure a cell with the reorder accessory, we will automatically put the collection in reordering mode when you press this accessory, assuming you implement the right callbacks.

similarly for other accessories.

[[advances in diffable datasources]]

outline disclosure -> cell automatically communicates with the datasource and expands / collapses the children of this cell

```swift
let cellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, String> { (cell, indexPath, item) in
    // @todo configure the cell's content
                                                                                            
    cell.accessories = [
        .disclosureIndicator(),
        .delete()
    ]
}
```

* smart system defaults
* flexible and customizable

The system knows that the disclosure indicator is always supposed to be trailing, whereas delete always appears on the leading.  So iOS automatically sorts and shows on the correct size.

System also knows that while disclosure is visible all the time, delete should only be visible in editing mode.  So UIKit automatically animates this.

Can also do `.displayed(.whenNotEditing)`.

