# Feature overview
You can think of this as a filter for the APIs that you call.  You can only get a subset of resources.
When user modifies their selection, your app is notified.
Will affect all apps using PhotoKit.  Even apps you've already shipped will be able to be put into this mode.

New "Select photos..." option in permission prompt
Allowing you to select individual photos.

Data visible to the app is limited to the user's selection.

How does user modify?
* settings
* User will be prompted when photos re accessed to modify selection.  We provide APIS to manage and suppress this prompt, but it's turned on by default.

## why?
Give users more control over their data.  We've seen photo libraries grow.  Users do not want to give 3rd parties access to the whole library.

Most apps don't need to get access to the full library.  What features can be added without access?

* Uploading a profile photo.
* Setting photos in a message / social media post?
* Embedding photos into a document

All these can be built with `PHPicker` and don't need any photo library access.

## PHPicker
* Replacement for `UIImagePickerController`
* Improved with search and multi-select
* Doesn't require photo access

[[Meet the new photos picker]]

## what's left?
* Saving an image from a message or post
* Simple camera apps

Can now do add-only access

[[What's new in Photos API]]

## full access

* browsing
* editing
* camera
* backup

# Limited access API
## querying for your application's authorization status

New authorization status value: `limited`
New enumeration: `PHAccessLevel`
* add only
* read-write

Now, you can check if your app is authorized for read/write vs add-only.  
Limited library doesn't affect add-only.


```swift
import Photos

//since we want to know if the user has authorized us for full access
//need to pass `.readWrite` here.
let accessLevel: PHAccessLevel = .readWrite
let authorizationStatus = PHPhotoLibrary.authorizationStatus(for: accessLevel)

switch authorizationStatus {
case .limited:
    print("limited authorization granted")
default:
    //FIXME: Implement handling for all authorizationStatus values
    print("Not implemented")
}
```
## requesting access
Request via fetch or change request.  We recommend relying on a user interaction.
Request authorization for access level

Keep in mind, the user only is prompted if the status is not already determined.

```swift
import Photos

let requiredAccessLevel: PHAccessLevel = .readWrite //or .addOnly, whatever your app needs
PHPhotoLibrary.requestAuthorization(for: requiredAccessLevel) { authorizationStatus in
    switch authorizationStatus {
    case .limited:
        print("limited authorization granted")
    default:
        //FIXME: Implement handling for all authorizationStatus
        print("Unimplemented")
        
    }
}
```

old APIs are "marked for future deprecation"
```swift
authorizationStatus() -> PHAuthorizationStatus
requestUathorization(_ handler: @escaping(PHAuthorizationStatus) -> Void)
```

These will return `.authorized` instead of `.limited`, even in the `limited` situation.

`PHAssets` created with `PHAssetCreationRequests` accessible to your app
Can't create or fetch user albums
No cloud shared assets or albums

# UI considerations
## prompt user to change selection
```swift
import PhotosUI

let library = PHPhotoLibrary.shared()
let viewController = self

library.presentLimitedLibraryPicker(from: viewController)
```

Monitor changes through `PHPhotoLibraryChangeObserver` update.  Updates may come if the user changes settings, if stuff is deleted in iCloud, etc.

You may want to add a button to pull up this screen.

## stop auto prompt on first access
Set `PHPhotoLibraryPreventAutomaticLimitedAccessAlert` in Info.plist.  "Prevent limtied photos access alert".  "This string will be in an updated version of Xcode"

# wrapup
* reconsider if your app needs photo library access.
* Adopt the new authorization APIs
* Integrate the limited library management UI into your app
* Stop the recurring selection prompt from appearing for your app

