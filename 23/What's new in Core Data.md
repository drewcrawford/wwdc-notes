Elevate your app's data persistence with improvements in Core Data. Learn how you can use composite attributes to create more intuitive data models. We'll also show you how to migrate your schema through disruptive changes, when to defer intense migrations, and how to avoid overhead on a person's device. To get the most out of this session, you should be familiar with handling different data types in Core Data as well as the basics of lightweight migration.

# Composite attributes
New type of attribute.  Encapsulates complex or custom data types.
Composed of built-in core data datatypes
Nesting Composite attributes is supported

Alternative to transformable type attributes.  Allow NSfetchRequests with NSPredicate configured with composite attribute namespaces keypath

Encapsulate flattened attributes.
Improve fetch performance by prefetching a relationship?  Effect of embedding composite attribute is it prevents faulting the remote entity.

NScompositeAttributedescription.
Attribute type is NSCompositeAttributeType.
`.elements` array contains the attributes
Must only contain `NSAttributeDescription`.  


###  5:39 - Adding a composite attribute
```swift
enum PaintColor: String, CaseIterable, Identifiable {
    case none, white, blue, orange, red, gray, green, gold, yellow, black
    var id: Self { self }
}

extension Aircraft {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<Aircraft> {
        return NSFetchRequest<Aircraft>(entityName: "Aircraft")
    }

    @NSManaged public var aircraftCategory: String?
    @NSManaged public var aircraftClass: String?
    @NSManaged public var aircraftType: String?
    @NSManaged public var colorScheme: [String: Any]?
    @NSManaged public var photo: Data?
    @NSManaged public var tailNumber: String?
    @NSManaged public var logEntries: NSSet?

}
```

###  5:53 - Setting a composite attribute
```swift
private func addAircraft() {
    viewContext.performAndWait {
        let newAircraft = Aircraft(context: viewContext)
        
        newAircraft.tailNumber = tailNumber
        newAircraft.aircraftType = aircraftType
        newAircraft.aircraftClass = aircraftClass
        newAircraft.aircraftCategory = aircraftCategory
        
        newAircraft.colorScheme = [
            "primary": primaryColor.rawValue,
            "secondary": secondaryColor.rawValue,
            "tertiary": tertiaryColor.rawValue
        ]
        
        do {
            try viewContext.save()
        } catch {
            // ...
        }
    }
}
```

###  6:11 - Fetching a composite attribute
```swift
private func findAircraft(with color: String) {
    viewContext.performAndWait {
        let fetchRequest = Aircraft.fetchRequest()
        fetchRequest.predicate = NSPredicate(format: "colorScheme.primary == %@", color)
        
        do {
            var fetchedResults: [Aircraft]
            fetchedResults = try viewContext.fetch(fetchRequest)
            
            // ...
        } catch {
            // Handle any errors that may occur
        }
    }
}
```

I guess a lot of this is stringly typed?

# Stage your migrations

Lightweight migration is the preferred fmethod of migration
Updates data storage to be in sync with data model
coredata has built-in migration tools

[[Evolve your Core Data Schema]]

Try a staged migration if you exceed lightweight migration
* migrate models with non-conforming lightweight changes
* simplify your app
* give your app execution control during migration

1.  identify models with non-conforming changes
2.  decompose the changes into stages
3. describe a total ordering of models to Core Data
4. have CD perform the migration

Manually review schema changes?
open the store with the lightweight migration options set?
`NSMIgratePersistentStoresAutomatically` and `NSInferMappingModelAutomaticallyOption`.  If they're not lightweight-eligible, you'll receive an error.

can use `NSMappingModel.inferredMappingModel(forSourceModel:destinationModel:)`, inferred model or nil.

basically we introduce new steps to make it migratable.  Series of migrations where ach model is migratable but equivalent to non-conforming migration.

so we copy data from aircraft entity into new flightdata entity, and relate those to the aircraft, kinda like an attachment paradigm.

in modelv3, old flightData attribute is deleted from the old entity.

each step is within the capabilities of lightweight migration.  To describe total orderin of models, we have these classes
`NSStagedMigrationManager`
`NSCustoMigrationStage`
`NSLIghtweightMigrationStage`
`NSManagedObjectModelReference`

manager -> manages and applies the stages as a total ordering
manages the event loop
provides access to the migrating store via NSPersistentContainer.
added with NSPersistentStoreStagedMigrationManagerOptionKey.

form the basis for moving between models.
Create a migration stage for each model change

NSLightweightMigrationStage -> don't require decomposition, lightweight eligible
hopefully the majority of your models
supplement the total ordering of models.
include all versions

each decomposed version of model uses NSCustomMigrationStage.  And contain a source model reference and a destination model reference.
Optional `willMigrateHandler` and `didMigrateHandler`

stage migrations make use of NSManagedObjectMOdelReference.
Promise of NSManagedObjectModel.  CD will full this promise.
Initialize with version checksum.
Checksum can be obtained using versionChecksum() method.
Output in the xcode build log under 'compile data model'.  Search for 'version checksum'
contained in the `VersionInfo.plist` for models.


###  16:00 - Creating managed object model references for staged migration
```swift
let v1ModelChecksum = "kk8XL4OkE7gYLFHTrH6W+EhTw8w14uq1klkVRPiuiAk="
let v1ModelReference = NSManagedObjectModelReference(
    modelName: "modelV1"
    in: NSBundle.mainBundle
    versionChecksum: v1ModelChecksum
)

let v2ModelChecksum = "PA0Gbxs46liWKg7/aZMCBtu9vVIF6MlskbhhjrCd7ms="
let v2ModelReference = NSManagedObjectModelReference(
    modelName: "modelV2"                          
    in: NSBundle.mainBundle                                                 
    versionChecksum: v2ModelChecksum
)

let v3ModelChecksum = "iWKg7bxs46g7liWkk8XL4OkE7gYL/FHTrH6WF23Jhhs="
let v3ModelReference = NSManagedObjectModelReference(
    modelName: "modelV3"
    in: NSBundle.mainBundle
    versionChecksum: v3ModelChecksum
)
```

###  16:19 - Creating migration stages for staged migration
```swift
let lightweightStage = NSLightweightMigrationStage([v1ModelChecksum])
lightweightStage.label = "V1 to V2: Add flightData attribute"

let customStage = NSCustomMigrationStage(
    migratingFrom: v2ModelReference,
    to: v3ModelReference
)

customStage.label = "V2 to V3: Denormalize model with FlightData entity"
```

###  16:54 - willMigrationHandler and didMigrationHandler of NSCustomMigrationStage
```swift
customStage.willMigrateHandler = { migrationManager, currentStage in
    guard let container = migrationManager.container else {
        return
    }

    let context = container.newBackgroundContext()
    try context.performAndWait {
        let fetchRequest = NSFetchRequest<NSFetchRequestResult>(entityName: "Aircraft")
        fetchRequest.predicate = NSPredicate(format: "flightData != nil")
            
        do {
           var fetchedResults: [NSManagedObject]
           fetchedResults = try viewContext.fetch(fetchRequest)
           
           for airplane in fetchedResults {
                let fdEntity = NSEntityDescription.insertNewObject(
                    forEntityName: "FlightData,
                    into: context
                )
             
                let flightData = airplane.value(forKey: "flightData")
                fdEntity.setValue(flightData, forKey: “data”)
                fdEntity.setValue(airplane, forKey: "aircraft")
                airplane.setValue(nil, forKey: "flightData")
            }
            try context.save()
        } catch {
            // Handle any errors that may occur
        }
    }
}
```

"It's possible that the aircraft class may not exist during the migration" ??

###  17:41 - Loading the persistent stores with an NSStagedMigrationManager
```swift
let migrationStages = [lightweightStage, customStage]
let migrationManager = NSStagedMigrationManager(migrationStages)

let persistentContainer = NSPersistentContainer(
    path: "/path/to/store.sqlite",
    managedObjectModel: myModel
)

var storeDescription = persistentContainer?.persistentStoreDescriptions.first

storeDescription?.setOption(
    migrationManager,
    forKey: NSPersistentStoreStagedMigrationManagerOptionKey
)

persistentContainer?.loadPersistentStores { storeDescription, error in
    if let error = error {
        // Handle any errors that may occur
    }
}
```

persistent stores are loaded to affect migration process.  CD automatically apply required stages and migrate store schema.

# Defer your migrations
some migrations require additional runtime that you can't do in fg.
migration can take time
Long migrations can frustrate users.
defer some work during lightweight migration tasks
during lightweight migration, if an entity has a migration transformation requiring cleanup, such as updating indices or dropping column, table transformation can be delayed until you deem that the resources are available.

Still synchronous and occurs normally, only the cleanup of schema is preferred.

`NSPersistentStoreDeferredLightweightMigrationOptionKey`.

backwards compatible with macOS big sur and iOS 14

only available for sqlite store types!

examples
* removing attributes or relationships
* changing relationships where an entity hierarchy no longer exists
* changing relationships from ordered to non-ordered

check persistent store's metadata.  If it contains the key, that is a signal to you that there is deferred work tbd.
Can use `finishDeferredLightweightMigration` to finish.




###  21:01 - Adding a persistent store with NSPersistentStoreDeferredLightweightMigrationOptionKey option
```swift
let options = [
    NSPersistentStoreDeferredLightweightMigrationOptionKey: true,
    NSMigratePersistentStoresAutomaticallyOption: true,
    NSInferMappingModelAutomaticallyOption: true
]

let store = try coordinator.addPersistentStore(
    ofType: NSSQLiteStoreType,
    configurationName: nil,
    at: storeURL,
    options: options
)
```

###  21:17 - Executing deferred migrations
```swift
// After using BGProcessingTask to run migration work   
let metadata = coordinator.metadata(for: store)
if (metadata[NSPersistentStoreDeferredLightweightMigrationOptionKey] == true) {
    coordinator.finishDeferredLightweightMigration()
}
```

consider using background task api to schedule completion.  BGProcessingTask.

consider designing stages that take advantage of both api's capabilities.

# Wrap up
* wrap data types with composite attributes
* stage complex model migrations
* improve app performance with deferred migrations

[[Evolve your Core Data Schema]]

# Resources
* https://developer.apple.com/documentation/coredata
* https://developer.apple.com/documentation/coredata/migrating_your_data_model_automatically
