#CloudKit 

# Swift concurrency
Framework that gives your application access to a database on iCloud.  CKContainer, through which you can access multiple CKDatabase.
Each container has a public db.  If you're logged into icloud, you have a private db.  Sharing to the current iCloud user in the shared db.

* Exposed on CKContainer and CKDatabase
* Ease onboarding to CloudKit.framework
* Optimized for UI applications

## Existing APIs
Operation API
* NSOperation subclasses
* Fulyl configurable
* Features beyond container and database API

* Batching
* Paging
* Grouping

## Swift concurrency
* Async functions
* Per-item callbacks using Swift.Result
* Improved container and database APIs

[[Meet asyncawait in Swift]]

* Specific to the container and database function API

```swift
// Sample code using existing Convenience API

/// Delete the last person record.
/// - Parameter completionHandler: An optional handler to process completion `success` or `failure`.
func deleteLastPerson(completionHandler: ((Result<Void, Error>) -> Void)? = nil) {
    database.delete(withRecordID: lastPersonRecordId) { recordId, error in
        if let recordId = recordId {
            os_log("Record with ID \(recordId.recordName) was deleted.")
        }
        if let error = error {
            self.reportError(error)
            // If there is a completion handler, pass along the error here.
            completionHandler?(.failure(error))
        } else {
            // If there is a completion handler, like during tests, call it back now.
            completionHandler?(.success(()))
        }
    }
}
```

```swift
// Sample code updated to CloudKit Async API

/// Delete the last person record.
func deleteLastPerson() async throws {
    do {
        let recordId = try await database.deleteRecord(with: lastPersonRecordId)
        os_log("Record with ID \(recordId.recordName) was deleted.")
    } catch {
        self.reportError(error)
        throw error
    }
}
```

Each of our code samples have updates which demonstrate how the code can be refactored to use Swift concurrency.

## Per-item callbacks
Here's a CKFetchRecords operation that gets back 4 CKRecord payloads.

1.  Success
2.  Operation-wide error, whole operation fails
3.  Partial failure

```swift
// Error reporting in CKFetchRecordsOperation

extension CKFetchRecordsOperation {
    var perRecordCompletionBlock: ((CKRecord?, CKRecord.ID?, Error?) -> Void)?

    var fetchRecordsCompletionBlock: (([CKRecord.ID : CKRecord]?, Error?) -> Void)?
}


fetchRecordsOp.perRecordCompletionBlock = { record, recordID, error in
    // error is CKError.unknownItem. 
}

fetchRecordsOp.fetchRecordsCompletionBlock = { recordsByRecordID, operationError in
    // operationError is CKError.partialFailure.
    // operationError.partialErrorsByItemID[missingRecordID] is CKError.unknownItem.
}
```

Code expects a per-item error twice.  Once as a toplevel error, once as a per-item callback.

Successes are also doubled.  First, as a top-level parameter to per-item callback, and another in `partialErrorsByItemID`.


```swift
// Error reporting in CKFetchRecordsOperation

extension CKFetchRecordsOperation {
    var perRecordResultBlock: ((CKRecord.ID, Result<CKRecord, Error>) -> Void)?

    var fetchRecordsResultBlock: ((Result<Void, Error>) -> Void)?
}


fetchRecordsOp.perRecordResultBlock = { recordID, result in
    // result is .failure(CKError.unknownItem) or .success(record).
}

fetchRecordsOp.fetchRecordsResultBlock = { result in
    // result is .success.
}
```

Result is now strongly typed.

One block is for per-item reporting, another is for operation reporting.  

Surface separate per-item and per-operation callbacks everywhere.  

All CKOperations now expose per-item callbacks.

## Improved container/database APIs
* build upon existing function API approach
* Adds functionality previously present in operation API
* Leverages default parameters and `Swift.Result`
* Has callback and async variants

New functionality
* Batching
* Paging
* Fetching changes
* Grouping
* Configurability

```swift
// Batched modifications

func deleteLastPeople() async throws {
    do {
        let recordIds = [lastPersonRecordId, penultimatePersonRecordId]
        let (_, deleteResults) = try await database.modifyRecords(deleting: recordIds)
        for (recordId, deleteResult) in deleteResults {
            switch deleteResult {
            case .failure(let error):
                self.reportError(error, itemId: recordId)
            case .success:
                os_log("Record with ID \(recordId.recordName) was deleted.")
            }
        }
    } catch let operationError {
        self.reportError(operationError)
        throw operationError
    }
}
```

Similar examples for each feature in sample code.

# Encrypted fields
1. How CK protects user data?
2. New data encryption feature
3. Prerequisites to use

## Data protection
* Account-based protection
	* CK backed-apps, all apple CK-backed apps.  Only authorized users can access the data.
	* Private and shared database only (not public database)
* Cryptographic protection
	* Apple apps and services data
	* All of your CKAsset data
	* Protected by the user's keychain
	* Compatible with sharing functionality
	* Private and shared databases only.
	* Used for data that is sensitive or private

This year, cryptographic protection now available for all data types.

### Data encryption
```swift
extension CKRecord {
    @NSCopying open var encryptedValues: CKRecordKeyValueSetting { get }
}
```
1.  CK encrypts locally
2.  Sent to server
3.  Retrieve records
4.  Decrypted on new client

```swift
// Device 1: Encrypt data before calling CKModifyRecordsOperation.

myRecord.encryptedValues["encryptedStringField"] = "Sensitive value"

// Device 2: Decrypt data after calling CKFetchRecordsOperation.

let decryptedString = myRecord.encryptedValues["encryptedStringField"] as? String
```

Encrypt any CKRecord types except CKReference (needs to be visible to the server)

CKAsset is encrypted by default

[[Meet CloudKit Console]]

* visualize your encrypted fields through CloudKit Console

In console, encrypted types have the "Encrypted" prefix.

## Prerequisites for encryption
1.  Check account status
	1.  `open func accountStatus(completionHandler: @escaping (CKAccountStatus, Error?) -> Void)`
	2.  Need to be `available`.  Now have a `temporarilyUnavailable` state as well.  This indicates an account is open but not ready.
2.  Listen to `CKAccountChanged` notification for update

# Zone sharing
## Review
* `CKShare` abstracts share management from shared data
* Cryptographic access
* System-provided UI: `UICloudSharingController` etc.
* Custom UI: `CKFetchShareParticipantsOperation`

## Folder sharing example
Tree structure sharing.

`CKRecord.parent` references.  CK will treat hierarchy as a single shareable unit.

Makes parent references special.

Using folder as the group record mean that it will share teh whole folder.  Records added or removed from the hierarchy int he future are shared or unshared automatically.

```swift
// Share a record hierarchy


let zone = CKRecordZone(zoneName: "MyZone")

// Save zone...

let fileRecordA = CKRecord(recordType: "File", recordID: CKRecord.ID(zoneID: zone.zoneID))
let fileRecordB = CKRecord(recordType: "File", recordID: CKRecord.ID(zoneID: zone.zoneID))
let folderRecord = CKRecord(recordType: "Folder", recordID: CKRecord.ID(zoneID: zone.zoneID))

fileRecordA.setParent(folderRecord)
fileRecordB.setParent(folderRecord)

// Save records...
```

```swift
// Share a record hierarchy


let share = CKShare(rootRecord: folderRecord)

do {
    let (saveResults, _) = try await database.modifyRecords(saving: [folderRecord, share])
    for (recordID, saveResult) in saveResults { 
        // Handle per-record result.
    }
} catch let operationError { 
    // Handle operation error.
}
```

CK can support multiple shares within the same zone as long as hirerachies don't overlap.

## Distinct types example
Ideally, you mark an entire zone as shared without manipulating any records within it.

Now, with zone sharing you can do that.

```swift
// Share a record zone


let zone = CKRecordZone(zoneName: "MyZone")

// Save zone... 

let share = CKShare(recordZoneID: zone.zoneID)

do {
    let (saveResults, _) = try await database.modifyRecords(saving: [share])
    for (recordID, saveResult) in saveResults { 
        // Handle per-record result.
    }
} catch let operationError { 
    // Handle operation error.
}
```

New, zone-wide share records

## Zone sharing
* Single CKShare record shares all records in zone
	* `CKRecordNameZoneWideShare`
* No parent references are required
* Cannot coexist with hierarchical shares
	* `CKRecordZoneCapabilityZoneWideSharing`
* All existing post-share mechanics are supported
* CKShareMetadata
	* `hierarchicalRotoRecordID` is `nil`
	* `rootRecord` is `nil`
* CKFetchShareMetadataOperation properties are ignored
	* `shouldFetchRootRecord`
	* `rootRecordDesiredKeys`

We already leverage zone sharing for homekit secure video, and homepod multiuser.


# Wrap up
* Swift concurrency
* Encrypted fields
* Zone sharing

[[Explore CloudKit]]


* https://github.com/apple/cloudkit-sample-encryption
* https://github.com/apple/cloudkit-sample-privatedb
* https://developer.apple.com/documentation/cloudkit

