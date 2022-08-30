Access photos and videos on iOS.  Out of process, don't need any library access.  Intuitive UI and eas-to-use API.

see previous sessions
[[Improve access to Photos in your app]]
[[Meet the new photos picker]]

```swift
var configuration = PHPickerConfiguration()
configuration.filter = .images
configuration.selection = .ordered
configuration.selectionLimit = 10

let picker = PHPickerViewController(configuration: configuration)
```

Overview of new features we added to picket.
Platform support

# New features
`PHPickerFilter` filter images, videos, and livePhotos.  May have other requirements, e.g. screenshot-stitching apps wants toonyl show screenshots.  now we added a filter for that.
screenRecordings, slomoVideos, cinematicVideos, etc.  Many, many more.

Createa  new filter using `PHAsset.PlaybackStyle`.  All new filters are backported, if you're targeting iOS 15, you can still use them as long as you have the 16 SDK.

For custom filters can use `all` and `not`.  Also backported.

```swift
var configuration = PHPickerConfiguration()

// iOS 15
// Shows videos and Live Photos
configuration.filter = .any(of: [.videos, .livePhotos])

// New: iOS 15
// Shows screenshots only
configuration.filter = .screenshots

// New: iOS 15
// Shows images excluding screenshots
configuration.filter = .all(of: [.images, .not(.screenshots)])

// New: iOS 16
// Shows cinematic videos
configuration.filter = .cinematicVideos
```

## Sheet presentation improvements
in iOS 15 we added half-ehight mode.  But some cases may implement a custom UI to adjust selecte dassets and have changes reflected back in the picker.  In iOS 16, you can deselect assets using their identifiers.  call `.deselectAsesets` or `.moveAssets`

# Platform support
System photos picker can only be used by iOS and iPadOS.  Thisy ear we're adding macOS and watchOS.  iPadOS picker is also updated with a new design.

iPadOS - sidebar.  
macOS - Also has a sidebar.  Multiple selection, fluid zooming, and search feature.  
NSOpenPanel - select assets fromt he system photo library.  For the first time, including those assets stored in iCloud photos.  Your app gets the new UI for free without any adoption work.

Drag and drop is supported in NSOpenPanel.  Also supported in standard picker on iPadOS, iOS, macOS.

When to use NSOpenPanel?  Great for simple flows.
Selected files may be removed periodically.
Copy selected files to your app directory

When to use PHPicker?
Great for media-centric macOS apps
You should also support NSOpenPanel.  Customers may still want to select asesets outside of photo libraries.

watchOS.  You can now have access to images on watch.  Runs out of process like iOS/macOS.  UI similar to iOS picker but smaller.  Browse photos in a grid or by collection.  Configure to show selection roder as well as specifying the selection limit.

* Images only
* 500 most recent iamges will be shown

AppKit api is very simialr to UIKit API.  It's an NSViewController.

It is time to move away from NSMediaLibraryBrowserController.  PHPickerViewController is much more powerful.  It's easier to maintain if you are working on UIKit and AppKit apps at the same time.

```swift
var configuration = PHPickerConfiguration()
configuration.filter = .images
configuration.selectionLimit = 10

let picker = PHPickerViewController(configuration: configuration)
```

## SwiftUI

```swift
struct ContentView: View {
    @Binding selection: [PhotosPickerItem]
    
    var body: some View {
        PhotosPicker(
            selection: $selection,
            matching: .images
        ) {
            Text("Select Photos")
        }
    }
}
```

Presented with only a few lines of SwiftUI code.  You hve access to picker API on all supported platforms.  Picker will automatically choose the best layout dependingo n platform, configuration, and screen space.  You can focus on making your app better.

## Loading photos and videos
Selection you receive through the swiftui binding only contains placeholder objects.  You still need to load actual asset data on demand.  Keep in mind that some asset data won't be loade di immediately.
* need to be downloaded
* need to be transcoded
errors are also possible.  
* inline loading UI is recommended

PhotosPicker uses Transferrable a new protocol for transferring data.

* define your own model objects that conform to `Transferable`
* Use `PhotosPickerItem` to load your model objects on demand
[[Meet Transferable]]

Dealing with lots of items or large assets.  May not be feasible to load everything in memory.
Use `FileTransferRepresentation` to reduce memory usage, to load assets as files.

When loading as files, your app is responsible for managing lifecycles
Copy picker provided image/video files to your app directory
Delete copied files that are no longer needed

## Demo
Works on iOS, macOS, and even watchOS.

## watchOS considerations
* designed for simple flows and short itneractions
* images are scaled based on device size
## family setup
Lets family members who don't have iPHone enjoy featuers and benefits of apple watch.
* iamges stored in iCloud photos can be selected
* The picker might show a loading UI before closing
Before you go, I just want to say we are committed to making system photos picker the best way for most apps to access photos and videos.  



# Frameworks













* https://developer.apple.com/documentation/photokit/selecting_photos_and_videos_in_ios
* https://developer.apple.com/documentation/photokit
