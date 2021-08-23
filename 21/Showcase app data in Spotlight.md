
# Core Data and Core Spotlight
* Showcase user content outside of your app
* Automatically updates Spotlight index
* Robust index management
* Tailor index results

## Benefits of delegate
* Feature pairty with Core Spotlight
* Less code required
* Additional fun features

```swift
let spotlightDelegate = NSCoreDataCoreSpotlightDelegate(forStoreWith: description,
                                                        coordinator: coordinator)
spotlightDelegate.startSpotlightIndexing()
```

Simple, easy-to-read and maintain.  Whod oesn't prefer less code?
# Setup
Decide what to index.

## Spotlight display name
`NSExpression` evaluated for each managed object
Controls "display name" of what shows up in spotlight results
Set in the Core Data model editor

## Create the spotlight delegate
Old initializer is deprecated

```swift
import Foundation
import CoreData

class CoreDataStack {
    private (set) var spotlightIndexer: TagsSpotlightDelegate?
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "Tags")

        guard let description = container.persistentStoreDescriptions.first else {
            fatalError("###\(#function): Failed to retrieve a persistent store description.")
        }

        description.type = NSSQLiteStoreType
        description.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
        description.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
                
        container.loadPersistentStores(completionHandler: { (_, error) in
            guard let error = error as NSError? else { return }
            fatalError("###\(#function): Failed to load persistent stores:\(error)")
        })
        
        spotlightIndexer = TagsSpotlightDelegate(forStoreWith: description,
                                                 coordinator: container.persistentStoreCoordinator)

        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        container.viewContext.automaticallyMergesChangesFromParent = true
        do {
            try container.viewContext.setQueryGenerationFrom(.current)
        } catch {
            fatalError("###\(#function): Failed to pin viewContext to the current generation:\(error)")
        }
        
        return container
    }()
}
```

No longer required to add an instance of the delegate to the store options prior to adding the store to the coordinator.

However, you must call `startSpotlightIndexing()`.

Store type must be sqllite
must have persistent history tracking enabled


# Customize
## Define a domain and index name
```swift
class TagsSpotlightDelegate: NSCoreDataCoreSpotlightDelegate {
    override func domainIdentifier() -> String {
        return "com.example.apple-samplecode.tags"
    }

    override func indexName() -> String? {
        return "tags-index"
    }
  
    override func attributeSet(for object: NSManagedObject) -> CSSearchableItemAttributeSet? {
        if let photo = object as? Photo {
            let attributeSet = CSSearchableItemAttributeSet(contentType: .image)
            attributeSet.identifier = photo.uniqueName
            attributeSet.displayName = photo.userSpecifiedName
            attributeSet.thumbnailData = photo.thumbnail?.data
            for case let tag as Tag in photo.tags ?? [] {
                if let name = tag.name {
                    if attributeSet.keywords != nil {
                        attributeSet.keywords?.append(name)
                    } else {
                        attributeSet.keywords = [name]
                    }
                }
            }
            return attributeSet
        } else if let object as? Tag {
            let attributeSet = CSSearchableItemAttributeSet(contentType: .text)
            attributeSet.displayName = tag.name
            return attributeSet
        }
        return nil
    }
}
```

Tell spotlight where to store the index data and better identify it later.  If you did not override the domain identifier, the default one is the store identifier.  If you do not override indexName, the default name is `nil`

## Define an attribute set
`CSSearchableItemAttributeSet`.
Metadata describing each indexed item

Number of predefined properties allowing you to specify the metadata being displayed.  Attributes you choose depend completely on your domain.  Can use predefined attributes or define your own.

**Not thread safe.**
```swift
class TagsSpotlightDelegate: NSCoreDataCoreSpotlightDelegate {
    override func domainIdentifier() -> String {
        return "com.example.apple-samplecode.tags"
    }

    override func indexName() -> String? {
        return "tags-index"
    }
  
    override func attributeSet(for object: NSManagedObject) -> CSSearchableItemAttributeSet? {
        if let photo = object as? Photo {
            let attributeSet = CSSearchableItemAttributeSet(contentType: .image)
            attributeSet.identifier = photo.uniqueName
            attributeSet.displayName = photo.userSpecifiedName
            attributeSet.thumbnailData = photo.thumbnail?.data
            for case let tag as Tag in photo.tags ?? [] {
                if let name = tag.name {
                    if attributeSet.keywords != nil {
                        attributeSet.keywords?.append(name)
                    } else {
                        attributeSet.keywords = [name]
                    }
                }
            }
            return attributeSet
        } else if let object as? Tag {
            let attributeSet = CSSearchableItemAttributeSet(contentType: .text)
            attributeSet.displayName = tag.name
            return attributeSet
        }
        return nil
    }
}
```

If your model indexes a relationship, `attributeSsetForObject` must be overridden

Since model is indexing tag objects, need to handle tags

## Define an indexing event loop


Can now call `stopSpotlightIndexing`. 

## Observe Index Update notifications
CoreData has added index update notifications.  Can be informed when update is complete (`NSCoreDataDCoreSpotlightDelegate.indexDidUpdateNotification`)
Contains 2 key-value pairs
* `NSStoreUUIDKey`
* `NSPersistentHistoryTokenKey`



```swift
class PhotosViewController: UICollectionViewController {
    @IBOutlet var generateDefaultPhotosItem: UIBarButtonItem!
    @IBOutlet var deleteSpotlightIndexItem: UIBarButtonItem!
    @IBOutlet var startStopIndexingItem: UIBarButtonItem!
    
    private var isTagging = false
    private var spotlightFoundItems = [CSSearchableItem]()
    private static let defaultSectionNumber = 0
    private var searchQuery: CSSearchQuery?
    var spotlightUpdateObserver: NSObjectProtocol?

    private lazy var spotlightIndexer: TagsSpotlightDelegate = {
        let appDelegate = UIApplication.shared.delegate as? AppDelegate
        return appDelegate!.coreDataStack.spotlightIndexer!
    }()
  
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // ...

        toggleSpotlightIndexing(enabled: true)
    }
  
    @IBAction func deleteSpotlightIndex(_ sender: Any) {
        toggleSpotlightIndexing(enabled: false)

        spotlightIndexer.deleteSpotlightIndex(completionHandler: { (error) in
            if let err = error {
                print("Encountered error while deleting Spotlight index data, \(err.localizedDescription)")
            } else {
                print("Finished deleting Spotlight index data.")
            }
        })
    }

    @IBAction func toggleSpotlightIndexingEnabled(_ sender: Any) {
        if spotlightIndexer.isIndexingEnabled == true {
            toggleSpotlightIndexing(enabled: false)
        } else {
            toggleSpotlightIndexing(enabled: true)
        }
    }

    private func toggleSpotlightIndexing(enabled: Bool) {
        if enabled {
            spotlightIndexer.startSpotlightIndexing()
            startStopIndexingItem.image = UIImage(systemName: "pause")
        } else {
            spotlightIndexer.stopSpotlightIndexing()
            startStopIndexingItem.image = UIImage(systemName: "play")
        }

        let center = NotificationCenter.default
        if spotlightIndexer.isIndexingEnabled && spotlightUpdateObserver == nil {
            let queue = OperationQueue.main
            spotlightUpdateObserver = center.addObserver(forName: NSCoreDataCoreSpotlightDelegate.indexDidUpdateNotification,
                                                         object: nil,
                                                         queue: queue) { (notification) in
                let userInfo = notification.userInfo
                let storeID = userInfo?[NSStoreUUIDKey] as? String
                let token = userInfo?[NSPersistentHistoryTokenKey] as? NSPersistentHistoryToken
                if let storeID = storeID, let token = token {
                    print("Store with identifier \(storeID) has completed ",
                          "indexing and has processed history token up through \(String(describing: token)).")
                }
            }
        } else {
            if spotlightUpdateObserver == nil {
                return
            }
            center.removeObserver(spotlightUpdateObserver as Any)
        }
    }
}
```

## Add deletion support
Previously you had to either implement CS API, or delete the entire graph in coredata.

New in iOS 15, you can call `.deleteSpotlightIndex` API.


# Validate
Define an extension for photos VC that uses the UISearchResultsUpdating protocol.

```swift
extension PhotosViewController: UISearchResultsUpdating {
    func updateSearchResults(for searchController: UISearchController) {
        guard let userInput = searchController.searchBar.text, !userInput.isEmpty else {
            dataProvider.performFetch(predicate: nil)
            reloadCollectionView()
            return
        }
        
        let escapedString = userInput.replacingOccurrences(of: "\\", with: "\\\\").replacingOccurrences(of: "\"", with: "\\\"")
        let queryString = "(keywords == \"" + escapedString + "*\"cwdt)"
        
        searchQuery = CSSearchQuery(queryString: queryString, attributes: ["displayName", "keywords"])

        // Set a handler for results. This will be a called 0 or more times.
        searchQuery?.foundItemsHandler = { items in
            DispatchQueue.main.async {
                self.spotlightFoundItems += items
            }
        }
        
        // Set a completion handler. This will be called once.
        searchQuery?.completionHandler = { error in
            guard error == nil else {
                print("CSSearchQuery completed with error: \(error!).")
                return
            }

            DispatchQueue.main.async {
                self.dataProvider.performFetch(searchableItems: self.spotlightFoundItems)
                self.reloadCollectionView()
                self.spotlightFoundItems.removeAll()
            }
        }

        // Start the query.
        searchQuery?.start()
    }
}
```

# Wrap up
* Find data in Spotlight with Core Data
* Easy to add to your project
* Advanced customization

https://developer.apple.com/documentation/coredata/nspersistentstorecoordinator/showcase_app_data_in_spotlight
