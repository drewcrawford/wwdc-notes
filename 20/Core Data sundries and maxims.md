#coredata
* view context
* background context


# Batch operations
* streamlined insert, update, and delete
* No save notifications posted
* No callbacks or accessor logic

But can work around with Persistent History
This provides notifications
```swift
storeDesc.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
```

## accelerated data population
`NSBatchInsertRequest`
* quick and seamless

```swift
//NSBatchInsertRequest.h

@available(iOS 13.0, *)
open class NSBatchInsertRequest : NSPersistentStoreRequest {
    open var resultType: NSBatchInsertRequestResultType

    public convenience init(entityName: String, objects dictionaries: [[String : Any]])
    public convenience init(entity: NSEntityDescription, objects dictionaries: [[String : Any]])

    @available(iOS 14.0, *)
    open var dictionaryHandler: ((inout Dictionary<String, Any>) -> Void)?
    open var managedObjectHandler: ((inout NSManagedObject) -> Void)?

    public convenience init(entity: NSEntityDescription, dictionaryHandler handler: @escaping (inout Dictionary<String, Any>) -> Void)
    public convenience init(entity: NSEntityDescription, managedObjectHandler handler: @escaping (inout NSManagedObject) -> Void)
}
```

Can pass in an array of dictionaries
Or a block to fill out a dictionary!
Greatly reduces peak memory ingestion.
```swift
//Earthquakes Sample - Regular Save

   for quakeData in quakesBatch {
        guard let quake = NSEntityDescription.insertNewObject(forEntityName: "Quake", into: taskContext) as? Quake else { ... }
        do {
            try quake.update(with: quakeData)
        } catch QuakeError.missingData {
            ...
            taskContext.delete(quake)
        }
        ...
    }
    do {
        try taskContext.save()
    } catch { ... }
```

vs

```swift
//Earthquakes Sample - Batch Insert

var quakePropertiesArray = [[String:Any]]()
for quake in quakesBatch {
    quakePropertiesArray.append(quake.dictionary)
}

let batchInsert = NSBatchInsertRequest(entityName: "Quake", objects: quakePropertiesArray)

var insertResult : NSBatchInsertResult
do {
    insertResult = try taskContext.execute(batchInsert) as! NSBatchInsertResult
    ... 
}
//Earthquakes Sample - Batch Insert with a block

var batchInsert = NSBatchInsertRequest(entityName: "Quake", dictionaryHandler: { 
    (dictionary) in
        if (blockCount == batchSize) {
            return true
        } else {
            dictionary = quakesBatch[blockCount]
            blockCount += 1
        }
    })
    var insertResult : NSBatchInsertResult
    do {
        insertResult = try taskContext.execute(batchInsert) as! NSBatchInsertResult
        ...
    }
```

1.  NSManagedObjectContext save took over 1 minute, 30mb.  Peak is json data.  
2.  Using `NSBatchInsert` with array of dictionaries use 25mb and 13sec. 
3.  With block ingestion, only 11sec.  

## unique constraints
Can now upsert.  `moc.mergePolicy = NSMergeByPropertyObjctTrumpMergePolicy`

## batch updates
Quick and easy to do mass property updates.
```swift
//Earthquakes Sample - Batch Update

let updateRequest = NSBatchUpdateRequest(entityName: "Quake")
updateRequest.propertiesToUpdate = ["validated" : true]
updateRequest.predicate = NSPredicate("%K > 2.5", "magnitude")

var updateResult : NSBatchUpdateResult
do {
    updateResult = try taskContext.execute(updateRequest) as! NSBatchUpdateResult
    ... 
}
```

## batch deletes
Easy to delete large portions fo the object graph
Relatoinship rules are observed
Great for expiration or TTL paradigms

```swift
// Batch Delete without and with a Fetch Limit

   DispatchQueue.global(qos: .background).async {
       moc.performAndWait { () -> Void in
          do {
              let expirationDate = Date.init().addingTimeInterval(-30*24*3600)

              let request = NSFetchRequest<Quake>(entityName: "Quake")
              request.predicate = NSPredicate(format:"creationDate < %@", expirationDate)

              let batchDelete = NSBatchDeleteRequest(fetchRequest: request)
              //this avoids long loncks on this context
			  batchDelete.fetchLimit = 1000
              moc.execute(batchDelete)
           }
       }
   }
```



# Tailored fetching
`managedObjectResultType`
Full object graph traversal
All the callbacks and accessor logic

We can use `fetchBatchSize` to only load a page-width.

Can trim down data.  `propertiesToFetch` and `relationshipKeyPathsForPrefetching`.  

`managedObjectIDResultType` -> thread safe, low overhead identifiers.

But what about in-between?  `dictionaryResultType`.  Can do complex data aggregation, etc.

```swift
//Fetch average magnitude of each place

let magnitudeExp = NSExpression(forKeyPath: "magnitude")
let avgExp = NSExpression(forFunction: "avg:", arguments: [magnitudeExp])

let avgDesc = NSExpressionDescription()
avgDesc.expression = avgExp
avgDesc.name = "average magnitude"
avgDesc.expressionResultType = .floatAttributeType

let fetch = NSFetchRequest<NSFetchRequestResult>(entityName: "Quake")
fetch.propertiesToFetch = [avgDesc, "place"]
fetch.propertiesToGroupBy = ["place"]
fetch.resultType = .dictionaryResultType
```

`countResultType`.  
# Driving with notifications
## ObjectID notifications
Generated from a context save
Lightweight save information
Vended from `NSPersistentHistoryTransaction`

```swift
//NSManagedObjectContext.h

@available(iOS 14.0, *)
extension NSManagedObjectContext {
    public static let willSaveObjectsNotification: Notification.Name
    public static let didSaveObjectsNotification: Notification.Name
    public static let didChangeObjectsNotification: Notification.Name
         
    public static let didSaveObjectIDsNotification: Notification.Name
    public static let didMergeChangesObjectIDsNotification: Notification.Name
}
```

Also added lots of notification keys.

## remote change notifications
Enable Persistent History with RCNs.
`NSRemoteChangeNotification.userInfo` contains most recent history token

```swift
//Enable Remote Change Notifications with Persistent History
storeDesc.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
storeDesc.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
```

### in-process
### out-of-process
## history pointers
Tailor history requests just like fetch requests, to avoid unnecessary info

```swift
let changeDesc = NSPersistentHistoryChange.entityDescription(with: moc)
let request = NSFetchRequest<NSFetchRequestResult>()

//Set fetch request entity and predicate
request.entity = changeDesc
request.predicate = 
    NSPredicate(format: "%K = %@",changeDesc?.attributesByName["changedObjectID"], objectID)
   
//Set up history request with distantPast and set fetch request              
let historyReq = NSPersistentHistoryChangeRequest.fetchHistory(after: Date.distantPast)
historyReq.fetchRequest = request
                    
let results = try moc.execute(historyReq)
```

# Wrap up
* Utilize batch operations
* Tailor fetches
* Use notifications instead of polling


