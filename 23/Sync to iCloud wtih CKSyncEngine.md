#icloud
Discover how CKSyncEngine can help you sync people's CloudKit data to iCloud. Learn how you can reduce the amount of code in your app when you let the system handle scheduling for your sync operations. We'll share how you can automatically benefit from enhanced performance as CloudKit evolves, explore testing for your sync implementation, and more. To get the most out of this session, you should be familiar with CloudKit and CKRecord types.

New API.  Designed to help sync data between device and cloud

# The state of sync
* sync is expected
* sync is not magic!
* simpler is often better

NKPersistentCloudKitContainer  -> local persistent
can now use CKSyncEngine
for more control, use CKDatabase and CKOperation

high-level APIs reduce complexity.  At its core, sync is mostly sending changes to one device and fetching them on another.

CKSyncEngine makes this smalelr and more focused, handle things specific to your app, etc.

NSPersistentCloudKitContainer has 70k lines of tests!!


# Meet CKSyncEngine
* convenient but flexible
* private and shared data
* backward compatible
* used by many apple apps
* `NSUbiquitousKeyValueStore`
	* rewritten on top of the sync engine!

have an existing engine?
* consider switching, not required
* less maintenance
* future enhancements
* smaller API
* submit feedback

application communicates with CKSyncEngine and hands it records and zones.  When the sync engine has work to do, it might batch.  Consults with system task scheduler.  Ensrues device is ready to sync.
Runs task against cloudkit server.  

sync engine batches changes.  Reduces memory overhead by not bringing records into memory until needed.  

## Scheduling
* automatic sync
* system conditions
* user experience vs device resources
* often fast, but can be delayed/deferred
* rely on automatic sync
* easier, more efficient
* manual sync when necessary
* testing with manual sync

# Getting started
## prerequisites
* `CKRecord` and `CKRecordZone`
* CloudKit capability
* Remote notifications capability

## starting
Initialize your `CKSyncEngine` on app launch
this starts listening for PNs and scheduling bg tasks.  Might happen at any time, and the sync engine needs to be initialized to handle.

Conform to `CKSyncEngineDelegate`.
`CKSyncEngine.State.Serialization`.
Handle `Event.stateUpdate`.  Persist this locally, to provide it the next time the process launches.

To help understand this, I'll explain with code examples.
### Initializing CKSyncEngine - 12:14
```swift
actor MySyncManager : CKSyncEngineDelegate {
    
    init(container: CKContainer, localPersistence: MyLocalPersistence) {
        let configuration = CKSyncEngine.Configuration(
            database: container.privateCloudDatabase,
            stateSerialization: localPersistence.lastKnownSyncEngineState,
            delegate: self
        )
        self.syncEngine = CKSyncEngine(configuration)
    }
    
    func handleEvent(_ event: CKSyncEngine.Event, syncEngine: CKSyncEngine) async {
        switch event {
        case .stateUpdate(let stateUpdate):
            self.localPersistence.lastKnownSyncEngineState = stateUpdate.stateSerialization
        }
    }
}
```

# Using CKSyncEngine
Syncing with the engine.

## send changes to server
Add changes in `CKSyncEngine.State`.  Alerts engine to schedule sync.
Ensures consistency and deduplicate changes.
Implement `nextRecordZoneChangeBatch`.  Calls to get the next batch of record changes to send to server.

Handle `sendDatabaseChanges` and `sentRecordZoneChanges`.  Posts once changes are sent.

### Sending changes to the server - 14:13
```swift
func userDidEditData(recordID: CKRecord.ID) {
    // Tell the sync engine we need to send this data to the server.
    self.syncEngine.state.add(pendingRecordZoneChanges: [ .save(recordID) ])
}

func nextRecordZoneChangeBatch(
    _ context: CKSyncEngine.SendChangesContext, 
    syncEngine: CKSyncEngine
) async -> CKSyncEngine.RecordZoneChangeBatch? {

    let changes = syncEngine.state.pendingRecordZoneChanges.filter { 
        context.options.zoneIDs.contains($0.recordID.zoneID) 
    }

    return await CKSyncEngine.RecordZoneChangeBatch(pendingChanges: changes) { recordID in
        self.recordToSave(for: recordID)
    }
}
```

## fetching changes from server
* CKSyncEngine automatically fetches
* Handle fetch events
	* fetchedDatabaseChanges
	* fetchedRecordZoneChanges
* handle optional fetch events
	* willFetchChanges
	* didFetchChanges
	* ex, may be useful to perform setup or cleanup tasks before or after fetching changes.

### Fetching changes from the server - 15:40
```swift
func handleEvent(_ event: CKSyncEngine.Event, syncEngine: CKSyncEngine) async {
    switch event {
        
    case .fetchedRecordZoneChanges(let recordZoneChanges):
        for modifications in recordZoneChanges.modifications {
            // Persist the fetched modification locally
        }

        for deletions in recordZoneChanges.deletions {
            // Remove the deleted data locally
        }

    case .fetchedDatabaseChanges(let databaseChanges):      
        for modifications in databaseChanges.modifications {
            // Persist the fetched modification locally
        }
      
        for deletions in databaseChanges.deletions { 
            // Remove the deleted data locally
        }

    // Perform any setup/cleanup necessary
    case .willFetchChanges, .didFetchChanges:
        break
      
    case .sentRecordZoneChanges(let sentChanges):

        for failedSave in sentChanges.failedRecordSaves {
            let recordID = failedSave.record.recordID

            switch failedSave.error.code {

            case .serverRecordChanged:
                if let serverRecord = failedSave.error.serverRecord {
                    // Merge server record into local data
                    syncEngine.state.add(pendingRecordZoneChanges: [ .save(recordID) ])
                }
            
            case .zoneNotFound: 
                // Tried to save a record, but the zone doesn't exist yet.
                syncEngine.state.add(pendingDatabaseChanges: [ .save(recordID.zoneID) ])
                syncEngine.state.add(pendingRecordZoneChanges: [ .save(recordID) ])
             
            // CKSyncEngine will automatically handle these errors
            case .networkFailure, .networkUnavailable, .serviceUnavailable, .requestRateLimited:
                break
              
            // An unknown error occurred
            default:
                break
            }
        }
      
    case .accountChange(let event):
        switch event.changeType {

        // Prepare for new user
        case .signIn:
            break
          
        // Delete local data
        case .signOut:
            break
          
        // Delete local data and prepare for new user
        case .switchAccounts: 
            break
        }
    }
}
```


## Handling errors
* automatically handles transient errors, network, throttling, account issues, etc
* automatic retry
* you handle application errors
* re-schedule sync to retry

you receive network errors for *awareness*, but sync engine automatically retries, don't worry about it.

You handle `accountChanges` events.  Happen at any time on a device.  Listens for changes, notify you to indicate signin, signout, or account switch information.  Your application will prepare for the change depending on the account type.  Sync engine will not begin syncing with icloud until there is an account present on the device.

Initialize a CKSyncEngine for each database
Existing CKShare invitation/acceptance flow

### Using CKSyncEngine with private and shared databases - 18:49
```swift
let databases = [ container.privateCloudDatabase, container.sharedCloudDatabase ]

let syncEngines = databases.map {
    var configuration = CKSyncEngine.Configuration(
        database: $0,
        stateSerialization: lastKnownSyncEngineState($0.databaseScope),
        delegate: self
    )
    return CKSyncEngine(configuration)
}
```

[[Get the most out of CloudKit sharing]]

# Testing and debugging
Automated tests are the best way to ensure stability.
Test the full 'device-to-device' flow.
Simulate specific edge cases
Set `automaticallySync = false`.


### Testing CKSyncEngine integration - 20:00
```swift
func testSyncConflict() async throws {
    
    // Create two local databases to simulate two devices.
    let deviceA = MySyncManager()
    let deviceB = MySyncManager()
    
    // Save a value from the first device to the server.
    deviceA.value = "A"
    try await deviceA.syncEngine.sendChanges()
    
    // Try to save the value from the second device before it fetches changes.
    // The record save should fail with a conflict that includes the current server record.
    // In this example, we expect the value from the server to win.
    deviceB.value = "B"
    XCTAssertThrows(try await deviceB.syncEngine.sendChanges())
    XCTAssertEqual(deviceB.value, "A")
}
```

## tips/tricks
* understand sequence of events
* logging record/zone IDs and timestamps
* write tests for user flows
* timestamps

# Wrap up
* download smple code
* apple/sample-cloudkit-sync-engine on github
* documentation
* send us feedback
* 
# Resources
* https://developer.apple.com/documentation/cloudkit/cksyncengine
* https://github.com/apple/sample-cloudkit-sync-engine
* 