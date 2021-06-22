#uikit 

# Performance fundamentals
```swift
// Structuring data

struct DestinationPost: Identifiable {
    // Each post has a unique identifier
    var id: String
    
    var title: String
    var numberOfLikes: Int
    var assetID: Asset.ID
}
```

Diffable datasource is populated with `id` properties, not destination itself.

```swift
// Setting up diffable data source

class DestinationGridViewController: UIViewController {
    // Use DestinationPost.ID as the item identifier
    var dataSource: UICollectionViewDiffableDataSource<Section, DestinationPost.ID>
    
    private func setInitialData() {
        var snapshot = NSDiffableDataSourceSnapshot<Section, DestinationPost.ID>()
        
        // Only one section in this collection view, identified by Section.main
        snapshot.appendSections([.main])
        
        // Get identifiers of all destination posts in our model and add to initial snapshot
        let itemIdentifiers = postStore.allPosts.map { $0.id }
        snapshot.appendItems(itemIdentifiers)
        
        dataSource.apply(snapshot, animatingDifferences: false)
    }
}
```

Identifier does not change.
Apply snapshot to datasource.

Previously, we did `reloadData` internally.  Now applying a snapshot in an animation will only apply differences.  `reconfigureItems` makes it easy to update contents of visible cells.

Create registration once for each cell.

```swift
// Cell registrations

let cellRegistration = UICollectionView.CellRegistration<DestinationPostCell,
                                                         DestinationPost.ID> {
    (cell, indexPath, postID) in

    let post = self.postsStore.fetchByID(postID)
    let asset = self.assetsStore.fetchByID(post.assetID)
    
    cell.titleView.text = post.region
    cell.imageView.image = asset.image
}
```

Properties used to configure cell.

```swift
// Cell registrations

let cellRegistration = UICollectionView.CellRegistration<DestinationPostCell,
                                                         DestinationPost.ID> {
    (cell, indexPath, postID) in
    ...
}
   
let dataSource = UICollectionViewDiffableDataSource<Section.ID,
                                                    DestinationPost.ID>(collectionView: cv){
    (collectionView, indexPath, postID) in
  
     return collectionView.dequeueConfiguredReusableCell(using: cellRegistration,
                                                           for: indexPath,
                                                          item: postID)
}
```

Note how registration is created outside the provider, and used inside.  This is important for performance, because if we create a registration outside, collectionview would never re-use any of its cells!

## Cell lifecycle
1.  Preparation
2.  Display

1.  Asks for cell from datasource.  For diffable, runs cell provider.  When the provider runs, CV is asked to dequeue using registration.  If it exists in the reuse queue, calls `prepareForResuse`.  If queue is empty, will init a new cell.
2.  Registration configure cell.  Configured cell is sent to CV
3.  CV: sizing and layout.  Now cell si ready to display

Display
1.  `willDisplayCell`
2.  cell is visible
3.  `didEndDisplayingCell` (when offscreen)
4.  Now back in reuse pool

Interruptions during scorlling are called 'hitches'.

## Producing a frame

1.  Touches are delivered
2.  Updates properties of views and layers. e .g. `contentOffset`.
3.  Layout.  Commit.
4.  Sent to the render server.  Each frame has a commit deadline.  Depends on refresh rate.

If you take too long, miss the deadline.  [[Explore UI animation hitches and the render loop]]

To avoid these hitches,

# Cell prefetching
Don't need a cell every frame.  Prefetching can give you some extra time.  It will be called when there's a short commit and extra time in the runloop.

Then using the prefetched cell will be quick.  Since we are able to get a head start, we're able to avoid a hitch.

1.  Commit for the frame.  No cells are needed.  Lots of time left.
2.  Recognizes the situation and uses the spare time to prefetch the next cell.
3.  Because the cell prefetch is expensive, it causes commit to start later than normal.  The commit still finishes before the deadline.

Up to 2x time for each cell.

Build with iOS 15 SDK.

## Title
* build with iOS 15 SDK
* Expanded in UICollectionView
	* Lists
	* Compositional layout using estimated sizes

New in UITableView
Improved performance and power

Use the extra time to run in a more energy-efficient state.  

## Lifecycle with prefetching

1.  Preparation.  Cell fully configured in this phase.  Don't wait for it to be visible.
2.  Prepared cell (waiting).
3.  Cell display

Two important considerations.

1.  Possible for a prepared cell to *never be displayed*.  This happens if the user changes scroll direction.
2.  Once the cell is displayed, it can go right back into the waiting state.  

Same cell can be used more than once for the same index path.  Cell will no longer be immediately added to the reuse pool.

On devices with higher framerate, still possible to have hitches during scrolling.

# Updating cell content
Updating existing cells.  How to display images with best possible performance using new APIs.

As we scroll the ap, cells are prepared offscreen.

We want to display images on remote server.  May not have the image to show.  When imageview is first visible, it will be blank and only filled in once server request completes.

```swift
// Existing cell registration

let cellRegistration = UICollectionView.CellRegistration<DestinationPostCell,
                                                         DestinationPost.ID> {
    (cell, indexPath, postID) in

    let post = self.postsStore.fetchByID(postID)
    let asset = self.assetsStore.fetchByID(post.assetID)
    
    cell.titleView.text = post.region
    cell.imageView.image = asset.image
}
```

Might need to be downloaded.  Asset object indicates this with `isPlaceholder`.  

Cells are used for different destinations, by the time the asset store returns, cell might be configured.

Instead, inform the collectionview datasource.  

`reconfigureItems` snapshot method.  Calling `reconfigureItems` will rerun registration handler.  Use this instead.  Uses the item's existing cell rather than dequeueing and configuring a new cell.

Method calls `reconfigureItems`.  

```swift
// Updating cells asynchronously

let cellRegistration = UICollectionView.CellRegistration<DestinationPostCell,
                                                         DestinationPost.ID> {
    (cell, indexPath, postID) in

    let post = self.postsStore.fetchByID(postID)
    let asset = self.assetsStore.fetchByID(post.assetID)
    
    if asset.isPlaceholder {
        self.assetsStore.downloadAsset(post.assetID) { _ in
            self.setPostNeedsUpdate(id: post.id)
        }
    }
    
    cell.titleView.text = post.region
    cell.imageView.image = asset.image
}
```

Keep view updating code in one place.

Can also use download asset method inside datasource.  Prefetching is a great place to kick off downloads.  Have it ready before the cell is visible.

## Image loading hitches
* Placeholders appear okay
* Full assets cause hitches on display
* Interrupts user scrolling

All images take time to decode.  Some images, like larger images, are too large to be decoded in time.

When configuration handler is first called and asset is a placeholder, code begins async request for full-size image.

When asset is downloaded later, the cell configuration handler is rerun with the final image.  When an imageview tries to commit a new image, it must prepare the image for display on MT.  This can take a long time.

Preparation is a mandatory process to display an image.  Render server can only display images that are bitmaps, e.g. they must be decompressed.  

Image views do this when it commit sa new image, on MT.  Ideally, we can prepare the image in advance and only update UI when complete.  That way, we dont' block MT.

## Image preparation API

`preparingForDisplay` `byPreparingForDisplay` 
`prepareForDisplay`.

ONly contains the pixel data that the renderer needs.  No additional work needed.

async ones run on an internal UIKit serial queue.  
```swift
// Using prepareForDisplay

// Initialize the full image
let fullImage = UIImage()

// Set a placeholder before preparation
imageView.image = placeholderImage

// Prepare the full image
fullImage.prepareForDisplay { preparedImage in
    DispatchQueue.main.async {
       self.imageView.image = preparedImage
    }
}
```

Prepared images solve a large problem in any image-heavy app.  

* Maintain small cache of prepared images
* Do not store on disk.  Save original assets instead.

Prefetching gives extra time to be downloaded and prepared.  Users will not see placeholder for long, and possibly not at all.

```swift
// Asset downloading – before image preparation

func downloadAsset(_ id: Asset.ID,
                   completionHandler: @escaping (Asset) -> Void) -> Cancellable {
  
    return fetchAssetFromServer(assetID: id) { asset in
        DispatchQueue.main.async {
            completionHandler(asset)
        }
    }
}
```

We can prepare the asset before calling ocmpletion handler.  These assets are large but also valuable.  So once image is prepared, we want to cache.  Image cache uses images size to estimate memory use.

```swift
// Asset downloading – with image preparation

func downloadAsset(_ id: Asset.ID,
                   completionHandler: @escaping (Asset) -> Void) -> Cancellable {
    // Check for an already prepared image
    if let preparedAsset = imageCache.fetchByID(id) {
        completionHandler(preparedAsset)
        return AnyCancellable {}
    }
    return fetchAssetFromServer(assetID: id) { asset in
        asset.image.prepareForDisplay { preparedImage in
            // Store the image in the cache.
            self.imageCache.add(asset: asset.withImage(preparedImage!))
            DispatchQueue.main.async {
                completionHandler(asset)
            }
        }
    }
}
```

Images can be large, and iOS 15 introduces a new API for preparing thumbnails.

`preparingThumbnail(of size:)` etc.
Image is read and processed with destination size in mind, saving CPU/memory time.

```swift
// Using prepareThumbnail

// Initialize the full image
let profileImage = UIImage(...)

// Set a placeholder before preparation
posterAvatarView.image = placeholderImage

// Prepare the image
profileImage.prepareThumbnail(of: posterAvatarView.bounds.size) { thumbnailImage in
    DispatchQueue.main.async {
        self.posterAvatarView.image = thumbnailImage
    }
}
```

Easier to accelerate image and avoid hitches.

## HIgh performance images
* Prepare iamges in advance
* Use placeholder images
* Reconfigure cells to update image view

# Next steps
* Download sample code
* Buidl with iOS 15 SDK
* Validate prefetching in table and collection views
* Adopt image prepartion and image resizing APIs

