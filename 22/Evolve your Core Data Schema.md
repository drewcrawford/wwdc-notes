#coredata 

Updating and migrating the core data schema.

# What is schema migration
* your app's data model may require changes
* Necessitates update of the storage schema

Without migrating your schema, your app won't work
core data willr efuse to open your persistent store
`NSPersistentStoreIncompatibleVersionHashError`

# Strategies for migration
* Core Data has built-in migration tools
* lightweight migration

This is the **preferred** method of migration
Automatically analyzes and infers the necessary migration changes
Performed at runtime
Changes have to fit a "pattern"

Attributes
* adding
* removing
* non-optional becoming optional
* Optional becomign non-optional and defining a default value
* renaming
	* set renaming identifier in destination model
	* xcode data model editor's property inspector

You can rename an attribute in v2 and rename again in v3.

Relationships
* adding
* removing
* renaming
* cardinality changes

Entities
* Adding
* Removing
* Renaming
* Creating new parent or child entities
* Moving attributes within the entity hierarchy
* Modifying the enitty hierarchy
* **Cannot** merge entity hierarchies.  If two existing entities do not share a common parent in source, they cannot in destination

Two options keys
* NSMigratePersistentStoresAutomaticallyOption
* NSInferMappingModelAutomaticallyOption
checked when adding a store to the coordinator
* NSPersistentContainer and NSPersistentStoreDescription has these keys set for you.

request lightweight migration if necessary, by setting these options to YES.  This is only needed if you're using certain apis.

```swift
import CoreData

let storeURL = NSURL.fileURL(withPath: "/path/to/store")
let momURL = NSURL.fileURL(withPath: "/path/to/model")
guard let mom = NSManagedObjectModel(contentsOf: momURL) else { 
    fatalError("Error initializing managed object model for URL: \(momURL)")
}
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: mom)
do {
    let opts = [NSMigratePersistentStoresAutomaticallyOption: true,
                      NSInferMappingModelAutomaticallyOption: true]

    try coordinator.addPersistentStore(ofType: NSSQLiteStoreType,
                                       configurationName: Optional<String>.none,
                                       at: storeURL,
                                       options: opts)
} catch {
    fatalError("Error configuring persistent store: \(error)")
}
```

make the changes to the data model int he same model file
No need to create a new version of the model to make changes.

`NSMappingModel.inferredMappingModel(forSoruceModel:destinationModel:)`
Returns valid model if coredata could migrate, otherwise nil.

Some changes exceed the capability of lightweight migration
Checking to see if this fits any capabilities of lightweight migration, it's discovered that it doesn't.

Can use **staged migration**.
Decompose the task into a series of migrations that are lightweight.  

A->B
A->A'->...->B

This results in a series of migrations where each model is now lightweight-migratable but equivalent to the nonconforming migration.

## Migration decomposition
1.  A
2. Add `tmpStorage`
3. Import data (separate from coredata)
4. Remove FLIGHT_DATA, rename tmpStorage to FLIGHT_DATA
5. Goal model (B)

Automate by serially migrating unprocessed models
App-specific logic needs to be restartable, in the event migration is interrupted by process termination.

# CloudKit schema migration
* requires a shared data model
* Model defined in Core Data
* used to generate CK schema
* deployed from development to production

limitations
* unique constraints aren't supported
* Undefined and objectID attribute types are unavailable
* All relationships must be optional and have an inverse
* Deny deletion rule is unsupported

schema can be modified in development
schema is immutable in production

CloudKit schema migration is more restrictive.
supported operations
* adding new fields
* adding new record types
	* cannot modify existing record types or fields!

Migration only changes the store file
Does not change the CloudKit schema
Run the schema initializer
Promote to production

Keep in mind, users will be using old versions.  Latest version of the app will know about new additions to the schema.  But old versions won't know about new fields or record types.

* CloudKit schema is additive
* consider the effects on older versions
* ex, forgetting to update old fields that old versions of your app use.

Strategies
* incrementally add new fields to existing record types.
	* older versions have access to every record, not every field
* version your entities
	* use a fetch request to fetch only records compatible with current app
	* older versions will effectively hide newer records
* Use a completely new container
	* Uploading a large dataset may take an extended period of time
* Test your app across different model versions

## Demo
`-com.apple.CoreData.SQLDebug 3`
`-com.apple.CoreData.MigrationDebug 3`

# Wrap up
* Use lightweight migration to update your data model
* Break down complex data model changes into multiple steps
* Consider your data omdel carefully if you use CloudKit

hope that helps
* https://developer.apple.com/forums/tags/wwdc2022-10120
* https://developer.apple.com/forums/create/question?&tag1=71&tag2=408030
* https://developer.apple.com/documentation/coredata/using_lightweight_migration

