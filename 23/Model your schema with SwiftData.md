#swiftdata 
Learn how to use schema macros and migration plans with SwiftData to build more complex features for your app. We'll show you how to fine-tune your persistence with @Attribute and @Relationship options. Learn how to exclude properties from your data model with @Transient and migrate from one version of your schema to the next with ease. To get the most out of this session, we recommend first watching "Meet SwiftData" and "Build an app with SwiftData" from WWDC23.

See [[Meet SwiftData]]
[[Build an app with SwiftData]]

powerful framework for data modeling and management.  Enhances your modern swift app.  Focuses entirely on code, noe xternal file formats, uses swift's new macro ssyetme to create a seamless API...

# Utilizing schema macros

## Original Trip model - 0:56
```swift
import SwiftUI
import SwiftData

@Model
final class Trip {
    var name: String
    var destination: String
    var start_date: Date
    var end_date: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

Just use `@Model`.

Default behavior is good, but I can fine tune it a little bit.



## Adding a unique attribute - 1:50
```swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    var start_date: Date
    var end_date: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

requires names to be unique.  If a trip already exists, then the persistent backend will update... to the latest values?  This is called an update.

unique constraints
* primitive value types: numeric, string, uuid
* can decorate relationships

## Specifying original property names - 2:48
```swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

Can preserve existing data and rename.  Avoid data loss.  Ensures my schema update will be a simple migration.

`@Attribute` can do much more:
* external data
* transformable
* spotlight integration
* hash modifier

swift data will implicitly discover inverses and set them for me.  No manaement required, just works.
## Cascading delete rule - 4:00
```swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    @Relationship(.cascade)
    var bucketList: [BucketListItem]? = []
  
    @Relationship(.cascade)
    var livingAccommodation: LivingAccommodation?
}
```

* tailor your `@Relationship` metadata
	* `originalname`
	* specify `toMany` count constraints
	* hash modifier

Decorate with `@Transient` - not persisted.  This hides properties from SwiftData.
Must provide a default value.  Ensures they have a logical value when fetched.


## Transient properties - 4:54
```swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    @Relationship(.cascade)
    var bucketList: [BucketListItem]? = []
  
    @Relationship(.cascade)
    var livingAccommodation: LivingAccommodation?

    @Transient
    var tripViews: Int = 0
}
```

see docs.

# Evolving schemas

Encapsulate your models at a specific version with `VersionedSchema`.  Each distinct version of your schema should be defined.
Order your versions with `SchemaMigrationPlan`.  This allows swiftdata to perform a migration, etc.

Define each migration stage.  Two different types of migration stages.
* lightweight.  
	* do not require any additional code to migrate.  Modifications like adding original names, or specifying the delete rules, are lightweight.
* custom
	* e.g. making the name of a trip unique.



## Defining versioned schemas - 7:12
```swift
enum SampleTripsSchemaV1: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        var name: String
        var destination: String
        var start_date: Date
        var end_date: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}

enum SampleTripsSchemaV2: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        @Attribute(.unique) var name: String
        var destination: String
        var start_date: Date
        var end_date: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}

enum SampleTripsSchemaV3: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        @Attribute(.unique) var name: String
        var destination: String
        @Attribute(originalName: "start_date") var startDate: Date
        @Attribute(originalName: "end_date") var endDate: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}
```

## Implementing a SchemaMigrationPlan - 7:49
```swift
enum SampleTripsMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [SampleTripsSchemaV1.self, SampleTripsSchemaV2.self, SampleTripsSchemaV3.self]
    }
    
    static var stages: [MigrationStage] {
        [migrateV1toV2, migrateV2toV3]
    }

    static let migrateV1toV2 = MigrationStage.custom(
        fromVersion: SampleTripsSchemaV1.self,
        toVersion: SampleTripsSchemaV2.self,
        willMigrate: { context in
            let trips = try? context.fetch(FetchDescriptor<SampleTripsSchemaV1.Trip>())
                      
            // De-duplicate Trip instances here...
                      
            try? context.save() 
        }, didMigrate: nil
    )
  
    static let migrateV2toV3 = MigrationStage.lightweight(
        fromVersion: SampleTripsSchemaV2.self,
        toVersion: SampleTripsSchemaV3.self
    )
}
```



## Configuring the migration plan - 8:40
```swift
struct TripsApp: App {
    let container = ModelContainer(
        for: Trip.self, 
        migrationPlan: SampleTripsMigrationPlan.self
    )
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

# Wrapup
* convey metadata with schema macros
* version schemas as your app evolves
[[Migrate to SwiftData]]
[[Dive deeper into SwiftData]]

# Resources
* https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app
* https://developer.apple.com/documentation/SwiftData
