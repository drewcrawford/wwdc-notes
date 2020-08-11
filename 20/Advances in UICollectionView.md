#uikit 
Expandable/collapsible outline

# APIs

## Originally
Separation of concerns
* Data -> `UICollectionViewDataSource`
* Layout -> `UICollectionViewLayout`, `UICollectionViewFlowLayout`
* presentation -> `UICollectionViewCell`, `UICollectionViewReusableCell`

## iOS 13

* Data -> DiffableDataSource
* Layout -> `UICollectionViewCompositionalLayout`
* presentation -> `UICollectionViewCell`, `UICollectionViewReusableCell`

## iOS 14

* Data -> DiffableDataSource + section snapshots
* Layout -> `UICollectionViewCompositionalLayout` + list configuration
* presentation -> `UICollectionViewCell`, `UICollectionViewReusableCell` + cell enhancements

### Diffable datasource enhancements

Automatic animations
No batch updates
[[advances in UI datasources (2019)]]

#### Section snapshots
Single section data
Composeable datasources
Hierarchical data, outline-style UIs

[[advances in diffable datasources]]

### Compositional Layout enhancements

Composeable
Fast
Section-specific layouts
Orthogonal scrolling

[[Advances in collectionview layout - 19]]

#### Lists

`UITableView`-like appearance
Swipe actions
Common cell layouts

Simple
Per-section or entire layout

```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let layout = UICollectionViewComposiionalLayout(...)
```

`UICollectionViewListCell`
header/footer support
sidebar appearance

[[Lists in UICollectionView]]

### Modern cells

#### Cell registrations
* Reusable
* Don't need to register cell class/nib
* Generic registration type

```swift
let reg = UICollectionView.CellRegistration<MyCell, ViewModel> { cell, indexPath, model in
	//configure cell's content from the view model
}

let dataSource = UICollectionViewDiffableDataSource<S,I>(collectionView: collectionView) { collectionView, indexPath, item -> UICollectionViewCell in
return collectionView.dequeueConfiguredReusableCell(using: reg, for: indexPath, item: item.viewModel)
}
```

#### Cell content configurations
Similar to `UITableViewCell` types
Use with any cell or even generic UIView

`UIListContentConfiguration.cell()`

[[Modern cell configuration]]

