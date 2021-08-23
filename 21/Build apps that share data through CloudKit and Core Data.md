#CloudKit 
#coredata 

# What is 'sharing'?
Kinda like sharing a document in google docs.  Single source of truth in the cloud.


# How do we share?
By far the most complicated feature we have built into NSPersistentCloudKidContainer.

private database and shared database.  Each mirrored to a persistent store in my app.  One MOC for both stores.

```swift
let privateStoreDescription = container.persistentStoreDescriptions.first!
let storesURL = privateStoreDescription.url!.deletingLastPathComponent()
privateStoreDescription.url = storesURL.appendingPathComponent("private.sqlite")
privateStoreDescription.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
privateStoreDescription.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)

let sharedStoreURL = storesURL.appendingPathComponent("shared.sqlite")
let sharedStoreDescription = privateStoreDescription.copy()
sharedStoreDescription.url = sharedStoreURL

let containerIdentifier = privateStoreDescription.cloudKitContainerOptions!.containerIdentifier
let sharedStoreOptions = NSPersistentCloudKitContainerOptions(containerIdentifier: containerIdentifier)
sharedStoreOptions.databaseScope = .shared
sharedStoreDescription.cloudKitContainerOptions = sharedStoreOptions
container.persistentStoreDescriptions.append(sharedStoreDescription)
```

```swift
@IBAction func shareNoteAction(_ sender: Any) {
  guard let barButtonItem = sender as? UIBarButtonItem else {
    fatalError("Not a UI Bar Button item??")
  }

  guard let post = self.post else {
    fatalError("Can't share without a post")
  }

  let container = AppDelegate.sharedAppDelegate.coreDataStack.persistentContainer
  let cloudSharingController = UICloudSharingController {
    (controller, completion: @escaping (CKShare?, CKContainer?, Error?) -> Void) in
    container.share([post], to: nil) { objectIDs, share, container, error in
			if let actualShare = share {
				post.managedObjectContext?.performAndWait {
					actualShare[CKShare.SystemFieldKey.title] = post.title
				}
			}
			completion(share, container, error)
		}
  }
  cloudSharingController.delegate = self

  if let popover = cloudSharingController.popoverPresentationController {
    popover.barButtonItem = barButtonItem
  }
  present(cloudSharingController, animated: true) {}
}
```

Evidently there is an application delegate notification for accepting invitations.

## Owners and participates
* Owners create and share objects
* Participants operate on shared objects

## Shared objects
* NSManagedObject <-> CKRecord
* NSPersistentcloudKitContainer uses record zone sharing
[[What's new in CloudKit]]

Typically manages a .private zone.  In Record Zone Sharing, shared CKRecords are contained inside a shared CKRecord Zone. 

NSPersistentCloudKitContainer manages these zones.  Because there's no root record, it has to understand how owners and participants apply to the entire record zone.

Designed to facilitate sharing for larger populations.  Each participant can access and operate on objects.  

IN my .private database, I see records/zones I own (whether or not they're shared).  e.g. I see private zones, and shared zones I own.  Others can modify records in the shared zones.

In .shared database, I see records others have shared with me.  

Another user will see a different set of zones in private and shared databases, depending on whether they own the zones.  

In many cases, CK can infer where records belong.  But you can tell to share in a specific shared zone.

How do we communicate what stuff is shared?  

## Applications change with sharing
* Decorate shared objects
* Conditionalize editing
* Display participants

Number of APIs to align with these concerns.
`fetchShares(matching objectIDs:)` lets me get CKShare for objects.

`canUpdateRecord`, `canDeleteRecord`, `canModifyManagedObjects` to customize UX.

Instead of invoking methods on NSPersistentCloudKitContainer directly, I built a protocol.  

One specific call site in main VC:  
```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(withIdentifier: "PostCell", for: indexPath) as? PostCell else {
        fatalError("###\(#function): Failed to dequeue a PostCell. Check the cell reusable identifier in Main.storyboard.")
    }
    let post = dataProvider.fetchedResultsController.object(at: indexPath)
    cell.title.text = post.title
    cell.post = post
    cell.collectionView.reloadData()
    cell.collectionView.invalidateIntrinsicContentSize()
    
    if let attachments = post.attachments, attachments.allObjects.isEmpty {
        cell.hasAttachmentLabel.isHidden = true
    } else {
        cell.hasAttachmentLabel.isHidden = false
    }
    
    if sharingProvider.isShared(object: post) {
        let attachment = NSTextAttachment(image: UIImage(systemName: "person.circle")!)
        let attributedString = NSMutableAttributedString(attachment: attachment)
        attributedString.append(NSAttributedString(string: " " + (post.title ?? "")))
        cell.title.text = nil
        cell.title.attributedText = attributedString
    }
    return cell
}
```

After adding these customizations, it was obvous I needed to ensure they work correctly.  

Protocol makes it easy to test these decision points by injection.

```swift
func testSharedPostsGetDisclosure() {
    var sharedObjectIDs: Set<NSManagedObjectID> = Set()
    let context = coreDataStack.persistentContainer.viewContext
    self.generatePosts(in: context, postSaveBlock: { posts in
        for (index, post) in posts.enumerated() where (index % 4) == 0 {
            sharedObjectIDs.insert(post.objectID)
        }
    })
    
    let provider = BlockBasedShareProvider(stack: coreDataStack)
    provider.isSharedBlock = sharedObjectIDs.contains
    mainViewController.sharingProvider = provider
    
    do {
        try mainViewController.dataProvider.fetchedResultsController.performFetch()
    } catch let error {
        XCTFail("Error while fetching \(error)")
    }
    
    reloadTableView()
    let rowCount = mainViewController.tableView(mainViewController.tableView,
                                                    numberOfRowsInSection: 0)
    XCTAssertEqual(100, rowCount)
    guard let expectedSharedImage = UIImage(systemName: "person.circle") else {
        XCTFail("Failed to get the person system image.")
        return
    }
    
    for index in 0..<rowCount {
        let indexPath = IndexPath(row: index, section: 0)
        let post = mainViewController.dataProvider.fetchedResultsController.object(at: indexPath)
        guard let title = post.title else {
            XCTFail("All posts should have been given a title.")
            return
        }
        
        guard let cell = mainViewController.tableView(mainViewController.tableView,
                                                           cellForRowAt: indexPath) as? PostCell else {
            XCTFail("Encountered an unexpected cell type in the main view controller's table view.")
            return
        }
        
        if sharedObjectIDs.contains(post.objectID) {
            guard let attributedText = cell.title.attributedText else {
                XCTFail("Failed to get the attributed text of \(cell). Was it not set?")
                return
            }
            
            guard let attachment = attributedText.attributes(at: 0, effectiveRange: nil)[.attachment] as? NSTextAttachment else {
                XCTFail("Expected an image attachment at the first character.")
                return
            }
            
            XCTAssertEqual(expectedSharedImage, attachment.image)
        } else {
            XCTAssertEqual(cell.title.text, title)
        }
    }
}

class BlockBasedShareProvider: SharingProvider {
    var coreDataStack: CoreDataStack
    init(stack: CoreDataStack) {
        coreDataStack = stack
    }
    
    func isShared(object: NSManagedObject) -> Bool {
        return isShared(objectID: object.objectID)
    }
    
    public var isSharedBlock: ((_ object: NSManagedObjectID) -> Bool)? = nil
    func isShared(objectID: NSManagedObjectID) -> Bool {
        guard let block = isSharedBlock else {
            return coreDataStack.isShared(objectID: objectID)
        }
        return block(objectID)
    }
    
    public var participantsBlock: ((_ object: NSManagedObject) -> [RenderableShareParticipant])? = nil
    func participants(for object: NSManagedObject) -> [RenderableShareParticipant] {
        guard let block = participantsBlock else {
            return coreDataStack.participants(for: object)
        }
        return block(object)
    }
    
    public var sharesBlock: ((_ objectIDs: [NSManagedObjectID]) -> [NSManagedObjectID: RenderableShare])? = nil
    func shares(matching objectIDs: [NSManagedObjectID]) throws -> [NSManagedObjectID: RenderableShare] {
        guard let block = sharesBlock else {
            return try coreDataStack.shares(matching: objectIDs)
        }
        return block(objectIDs)
    }
    
    public var canEditBlock: ((_ object: NSManagedObject) -> Bool)? = nil
    func canEdit(object: NSManagedObject) -> Bool {
        guard let block = canEditBlock else {
            return coreDataStack.canEdit(object: object)
        }
        return block(object)
    }
    
    public var canDeleteBlock: ((_ object: NSManagedObject) -> Bool)? = nil
    func canDelete(object: NSManagedObject) -> Bool {
        guard let block = canDeleteBlock else {
            return coreDataStack.canDelete(object: object)
        }
        return block(object)
    }
}
```

```swift
extension CoreDataStack: SharingProvider {
    func isShared(object: NSManagedObject) -> Bool {
        return isShared(objectID: object.objectID)
    }

    func isShared(objectID: NSManagedObjectID) -> Bool {
        var isShared = false
        if let persistentStore = objectID.persistentStore {
            if persistentStore == sharedPersistentStore {
                isShared = true
            } else {
                let container = persistentContainer
                do {
                    let shares = try container.fetchShares(matching: [objectID])
                    if nil != shares.first {
                        isShared = true
                    }
                } catch let error {
                    print("Failed to fetch share for \(objectID): \(error)")
                }
            }
        }
        return isShared
    }
    
    func participants(for object: NSManagedObject) -> [RenderableShareParticipant] {
        var participants = [CKShare.Participant]()
        do {
            let container = persistentContainer
            let shares = try container.fetchShares(matching: [object.objectID])
            if let share = shares[object.objectID] {
                participants = share.participants
            }
        } catch let error {
            print("Failed to fetch share for \(object): \(error)")
        }
        return participants
    }
    
    func shares(matching objectIDs: [NSManagedObjectID]) throws -> [NSManagedObjectID: RenderableShare] {
        return try persistentContainer.fetchShares(matching: objectIDs)
    }
    
    func canEdit(object: NSManagedObject) -> Bool {
        return persistentContainer.canUpdateRecord(forManagedObjectWith: object.objectID)
    }
        
    func canDelete(object: NSManagedObject) -> Bool {
        return persistentContainer.canDeleteRecord(forManagedObjectWith: object.objectID)
    }
}
```

1200 lines of test code.  

# Sensitive data
[[What's new in CloudKit]]
* New CKRecord.encryptedValues payload
* Encryption using the user's keychain
* One click adoption in Core Data

Tells NSPersistentCloudKid container that the value for this attribute should be stored in encrypted values payload

Or as code, `.allowsCloudEncryption`.  Use this to configure in your model code.

* CKRecord field types can only be set once
* Can't encrypt a field that is already encrypted, or vice versa
* Once schema is pushed to production, it is forever
* Remember to use `initializeSchema`

# Wrap up
* Lots of new API
* Updated sample application
* Updated documentation
* Test with NSPersistentCloudKidContainer
* File bugs with feedback assistant

* https://developer.apple.com/documentation/coredata/synchronizing_a_local_store_to_the_cloud

