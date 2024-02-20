Elevate your app's data persistence with improvements in Core Data. Learn how you can use composite attributes to create more intuitive data models. We'll also show you how to migrate your schema through disruptive changes, when to defer intense migrations, and how to avoid overhead on a person's device. To get the most out of this session, you should be familiar with handling different data types in Core Data as well as the basics of lightweight migration.

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
# Resources
* https://developer.apple.com/documentation/coredata
* https://developer.apple.com/documentation/coredata/migrating_your_data_model_automatically
