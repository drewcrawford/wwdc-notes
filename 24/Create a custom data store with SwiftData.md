Combine the power of SwiftData's expressive, declarative modeling API with your own persistence backend. Learn how to build a custom data store and explore how to progressively add persistence features in your app. To get the most out of this session, watch “Meet SwiftData” and “Model your schema with SwiftData” from WWDC23.

Use swiftdata with your own persistent backend.  New feature in swift data that allow you to use any document, file format, or persistence backend of your choice.

Work great with your existing swift data code.  Here, I can change the type of store just replacing model configuration with a JSONConfiguration.

# Overview

At a high level, store is responsible for fetching/storing all data required to implement persistent model.  

SampleTrips is built on the powerful synergy of swiftui and swiftdata.

In a typical app we have 3 parts
* swiftui - user interface, typically a view like a list of lable.
* Model -> model context
* Store -> model container

In this video we focus on ModelContainer (store).

We instantiate persistent models for each trip int he view.  Each trip has a corresponding persistent identifier uniquely identifying the model.  We track changes to save to store when needed.  

Initially, we have a temporary persistent identifier.  Store assigns a permanent persistent identifier.  Mapped to the former temporary identifier, aka remapping.

Store responds to the model context with the updated persistent ID.  After model context finishes updating its state, UI can update the view rendering the trip.

Role of the store:

* perform fetch and save
* Persist model values
* Communicate through `DataStoreSnapshot` instances

View communicates using models.  However, when model context communicates with store, it creates a snapshot to hold models.

Sendable, portable, version of models at a point in time.  Has persistent identifier.

Store consumes snapshots and applies the value to its storage.  When data is read, store creates a set of snapshots that align with PersistentModels we ask for.  The ModelContext creates Persistent Models for each snapshot.

# Meet DataStore

Stores play a critical role in SwiftData.  

3 key parts to the store
* configuration -> DataStoreConfiguration -> ModelConfiguration
* communication -> DataStoreSnapshot -> DefaultSnapshot
* implementation -> DataStore -> DefaultStore
(last column is default implementation)

Platforms best practices for performance and scalability.

## DataStore

* save
* fetch
* caching

Additional protocols
* history
* batch deletes

when fetching data from a store, we send DataStoreFetchRequest.  Once the store retrieves the model values, it creates a snapshot for each model and returns via fetch result.

DataStoreSavechangesResult.  
* remappedIdentifiers for newly updated models.

Model context processes save result, and updates its state, assigning new permanent persistent identifiers.
# Example store

* archival store - entire file is loaded when reading or writing
* array of snapshots

Begin implementing the two required methods.
* fetch
* save

The translation of a sort comparator can be involved process, instead we throw errors to make the model do it for us.

Includes remapped persistent identifiers.  Now that I have a complete custom data store, I can adopt in sample trip.  In app definition, I can change type of store by replacing model configuration with a JSONStoreConfiguration.


Implement a DataStore
* use any persistence backend
	* document
	* database
	* cloud storage

`ModelContext` filter and sort.

### Implement a JSON store - 8:15
```swift
// Implement a JSON store
@available(swift 5.9) 
@available(macOS 15, iOS 18, tvOS 18, watchOS 11, visionOS 2, *) 
final class JSONStoreConfiguration: DataStoreConfiguration {
    typealias StoreType = JSONStore
    
    var name: String
    var schema: Schema?
    var fileURL: URL
    
    init(name: String, schema: Schema? = nil, fileURL: URL) {
        self.name = name
        self.schema = schema
        self.fileURL = fileURL
    }
    
    static func == (lhs: JSONStoreConfiguration, rhs: JSONStoreConfiguration) -> Bool {
        return lhs.name == rhs.name
    }
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(name)
    }
}

@available(swift 5.9) 
@available(macOS 15, iOS 18, tvOS 18, watchOS 11, visionOS 2, *) 
final class JSONStore: DataStore {
    typealias Configuration = JSONStoreConfiguration
    typealias Snapshot = DefaultSnapshot
    
    var configuration: JSONStoreConfiguration
    var name: String
    var schema: Schema
    var identifier: String
    
    init(_ configuration: JSONStoreConfiguration, migrationPlan: (any SchemaMigrationPlan.Type)?) throws {
        self.configuration = configuration
        self.name = configuration.name
        self.schema = configuration.schema!
        self.identifier = configuration.fileURL.lastPathComponent
    }
    
    func save(_ request: DataStoreSaveChangesRequest<DefaultSnapshot>) throws -> DataStoreSaveChangesResult<DefaultSnapshot> {
        var remappedIdentifiers = [PersistentIdentifier: PersistentIdentifier]()
        var serializedTrips = try self.read()
        
        for snapshot in request.inserted {
            let permanentIdentifier = try PersistentIdentifier.identifier(for: identifier, entityName: snapshot.persistentIdentifier.entityName, primaryKey: UUID())
            let permanentSnapshot = snapshot.copy(persistentIdentifier: permanentIdentifier)
            serializedTrips[permanentIdentifier] = permanentSnapshot
            remappedIdentifiers[snapshot.persistentIdentifier] = permanentIdentifier
        }
        
        for snapshot in request.updated {
            serializedTrips[snapshot.persistentIdentifier] = snapshot
        }
        
        for snapshot in request.deleted {
            serializedTrips[snapshot.persistentIdentifier] = nil
        }
        
        try self.write(serializedTrips)
        
        return DataStoreSaveChangesResult<DefaultSnapshot>(for: self.identifier, remappedPersistentIdentifiers: remappedIdentifiers, deletedIdentifiers: request.deleted.map({ $0.persistentIdentifier }))
    }
    
    func fetch<T>(_ request: DataStoreFetchRequest<T>) throws -> DataStoreFetchResult<T, DefaultSnapshot> where T : PersistentModel {
        if request.descriptor.predicate != nil {
            throw DataStoreError.preferInMemoryFilter
        } else if request.descriptor.sortBy.count > 0 {
            throw DataStoreError.preferInMemorySort
        }
        
        let objs = try self.read()
        let snapshots = objs.values.map({ $0 })
        
        return DataStoreFetchResult(descriptor: request.descriptor, fetchedSnapshots: snapshots, relatedSnapshots: objs)
    }
    
    func read() throws -> [PersistentIdentifier: DefaultSnapshot] {
        if FileManager.default.fileExists(atPath: configuration.fileURL.path(percentEncoded: false)) {
            let decoder = JSONDecoder()
            decoder.dateDecodingStrategy = .iso8601
            let trips = try decoder.decode([DefaultSnapshot].self, from: try Data(contentsOf: configuration.fileURL))
            var result = [PersistentIdentifier: DefaultSnapshot]()
            
            trips.forEach { s in
                result[s.persistentIdentifier] = s
            }
            
            return result
        } else {
            return [:]
        }
    }
    
    func write(_ trips: [PersistentIdentifier: DefaultSnapshot]) throws {
        let encoder = JSONEncoder()
        encoder.dateEncodingStrategy = .iso8601
        encoder.outputFormatting = [.prettyPrinted, .sortedKeys]
        
        let jsonData = try encoder.encode(trips.values.map({ $0 }))
        try jsonData.write(to: configuration.fileURL)
    }
}
```

# Next steps
* adopt custom stores
* Implement support for any storage

[[What’s new in SwiftData]]
[[Track model changes with SwiftData history]]

# Resources
https://developer.apple.com/documentation/SwiftData