# Sync a Core Data store with the CloudKit public database
`NSPersistentCloudKitContainer` syncs Core Data stores with CloudKit
[[Using Core Data with CloudKit - 19]]

|         | Core Data              | CloudKit                      |
|---------|------------------------|-------------------------------|
| Objects | `NSManagedObject`      | `CKRecord`                    |
| Models  | `NSManagedObjectModel` | Schema                        |
| Stores  | `NSPersistentStore`    | `CKRecordZone` / `CKDatabase` |



# New API
(code listing not reproduced)
Need to add indexes against the public database.
Now for your app you’d have to repeat this for all of the record types.

* Use `NSPersistentCloudKidContainerOptions.databaseScope`
* Create recordName, modifiedAt indexes on all record types
	* May require schema initialization, see 2019 session
* Complete local mirror

Why do we want this?  

## Public database use case
* Data for everyone
* Data that you create
	* Templates
	* Initial dataset
* Data from your users
	* Commonly a high scores table.  You need all the data locally so you can quickly fetch/sort, and something your users contribute to
* Mix with Private Database as appropriate.

# Food for thought
## Special considerations for the public database
|         | Core Data              | CloudKit                      |
|---------|------------------------|-------------------------------|
| Objects | `NSManagedObject`      | `CKRecord`                    |
| Models  | `NSManagedObjectModel` | Schema                        |
| Stores  | `NSPersistentStore`    | `CKRecordZone` / `CKDatabase` |

## accounts/ownership in the private database
|        | Signed Out | Signed In |
|--------|------------|-----------|
| Read   | Can’t      | Can       |
| Create | Can’t      | Can       |
| Modify | Can’t      | Can       |

## Accounts/ownership in the *public* database
|        | Signed Out | Signed In |
|--------|------------|-----------|
| Read   | Can        | Can       |
| Create | Can’t      | Can       |
| Modify | Can’t      | ?         |

Simply ask for the current account, ask for the creator, compare.

But this is a ridiculous amount of code just to resolve one question mark.

Now we have `.canUpdateRecord(forManagedObjectWith:)`

If you want to power a larger view, like a table, can use `.canModifyObjects(in: <store>)` to figure out if **any** objects can be editable

# Importing data from the Public Database
Last year, we talked about importing in the private database based on push notifications.  

Import operation ->`CKFetchRecordZoneChangesOperation`.  Single request for all changes.  This relies on some technologies that are specific to the private database.
In the public database, we use `CKQueryOperation`.  We have to make one request per record type.  We also have to poll for changes rather than PN.

Only poll on application launch, or every 30 minutes.  This is to ensure that we align request load with usage.  However, qualify of freshness will be different.

Simpler models make fewer requests.  Restrict the entities for use in the public database, for those in use in the public store.  Don’t have random entities in there because then we will have to do more `CKQueryOperation`.

##  Deleting objects in the Public Database
If we delete an object in the private database, and export.  It leaves behind a tombstone.  This allows us to fetch the tombstone via `CKFetchRecordZoneChangesOperation`.

However, that’s not available in the public database.  So when we delete a record on one device, and export that delete to the public database, the record is deleted **immediately**.  When we execute `CKQueryOperation`, it says nothing has changed, because no tombstone is available for us to fetch.

Note that we advertise this, because `CKRecordZone` for the public DB does not support `fetchChanges`.  

Now we can use `canDeleteRecord(forManagedObjectWith:<Id>`.  If this returns false, then we can’t import the delete in the same way that it would for the private database.  It doesn’t mean that you **can’t delete**.  But there’s a difference between deleting data for the sake or removing it from the public database and removing it from the user interface in your application.

We just set a `delete` flag on the request.  Then we remove the trashed records from the user interface.  

Think about the UI state as distinct from whether something is on the database.

