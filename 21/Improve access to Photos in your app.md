# Updates to Photos picker
[[Meet the new photos picker]]

* Privacy
* Ordered selection
* Selection adjustment
* Reporting progress

## Privacy
We added a new section to settings to show one-time photo selection.

Replace your custom picker with the system one.

## Ordered selection
Can configure picker to show the selection order.
`.selectionLimit = 0`
`.selection = .ordered`

## Selection adjustment
What if people want to deselect photos using the picker?

Can now preselect photos.

Now your app can use the identifiers it got last time to preselect.  People can deselect them, or select additional photos.

We don't return the photo data for identifiers passed back?  Unclear to me

## When picker is completed
* All selected asets will be returned
* Only item providers for preselected assets will be empty

## When picker is cancelled
* Only preselected assets will be returned
* All item providers will be empty

`.preselectAssetIdentifiers = ...`

`newSelection[identifier] = existingSelection[identifier] ?? result`
## Reporting progress
May download assets from iCloud photos.

Previously, you can only show spinner which is not always great.  Now we give you progress.

# New cloud identifier APIs
Some apps require deeper access and integration.  Editing, camera, browsing.

PhotoKit provides a rich set of APIs for accessing/updating photos, vides, albums.  Unique identifiers that can be saved in your app and later used to retrieve the same records.

Every photo library and its identifiers are specific to the device.  Even when synced.  How to handle cross-device case?

New cloud identifier APIs.  Find same assets and albums between devices.

* Identify assets across devices
* Loookup local identifiers
* Available everywhere

* Simple way to identify assets
* Handle cloud complexity
* Even if account is signed out or running on a system that wasn't signed in.

Two kinds of identifiers
* Local
* Cloud => `PHCloudIdentifier`

## Cloud identifier mapping
You have local identifiers.  `PHPhotoLibrary.shared().cloudIdentifierMappings(forLocalIdentifiers: localIdentifiers)`

* Encode cloud identifiers
	* stringValue
* Share over network
	* CloudKit
* Up to you!  https://www.youtube.com/watch?v=T2_irPfYO_c

Look up library-specific local identifiers.  API for the other direction.

In a perfect world
* One-to-one mapping
* Not always simple
* Result type might have an error

Two types of errors to handle
* Photo library isn't able to resolve because the record isn't present or the app doesn't have access ot it, "Identifier Not Found" `errors.code == PHPhotosError.identifierNotFound`
* Multiple assets that match the provided cloud identifier.  Cloud state isn't in sync or the library relies on image content to find a match
	* `userInfo[PHLocalIdentifiersErrorKey]` and let the customer decide

* Use local identifiers for app interactions
* Perform lookup at load and save points
* Store and sync cloud identifiers

# Updates to limited library
Limited library selection is transparent.  Library you see will only contain the photos selected.

In iOS 14, limited library mode did not allow creation or access to custom album.  This year, we've added support.

With `presentLimitedLibraryPicker` API, I can pass a completion handler that gives me identifiers for added photos.

* Apps can create, update, and fetch their own albums
* New limtied library picker API with completion handler

# Looking forward
Assets library => deprecated in iOS 9

> We're planning to remove it in a future SDK

* New error codes

* https://developer.apple.com/documentation/photokit/selecting_photos_and_videos_in_ios
* https://developer.apple.com/documentation/photokit


