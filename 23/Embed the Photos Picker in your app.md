#photos

Discover how you can simply, safely, and securely access the Photos Library in your app. Learn how to get started with the embedded picker and explore the options menu and HDR still image support. We'll also show you how to take advantage of UI customization options to help the picker blend into your existing interface.

We recommend replacing your custom picker with the system one whenever possible

[[What's new in the Photos picker]]
[[Meet the new photos picker]]

# Embedded picker
It is really running in a separate process rendered on top of your app.

Photos picker is out of process.
Only what the user actually selects is passed back.

This year we're bringing more configuration options.

`.photosPickerDisabledCapabilities` lets you disable certain capabilities.
cancel/add can be hidden for example.
hide accessory UI around content ex navigation bar, toolbar
frame/size.  Using standard SwiftUI modifiers.

Can now select `selectionBehavior:continuous` to get live updates.
`photosPickerStyle(.inline)` to embed into your app, instead of presenting as a separate sheet.  even though it's embeded, it's still rendered in a separate process.

We have a prompt that displays on first use that explains the app can only access selectedp hotos.


Privacy settings is updated with more detailed explanation
[[23/What's new in Privacy|What's new in Privacy]]

Embedded picker - uses `.photosPickerStyle` modifier
To place your own UI around, use `.photosPickerAccessoryVisibility`.  Can control around specific edges.
`.photosPickerDisabledCapabilities` to implement your own features
`selectionBehavior: .continuous` to respond to updates in real time.

on iOS - top accessory is nav bar, bottom is toolbar.
iPadOS/macOS / sidebar on left, top/bottom similar to iOS.

## picker capabilities
search bar will be hidden if search capabilitiy is disabled.
If collectionnavigation is disabled, we disable that.  
Can also disable `stagingArea` (toolbar will be replaced with status label)
disabled `selectionActions` only the cancel button will be hidden, and add will still be visible.  This is to ensure you can select something.
With `.continuous` we remove add.

## styles
`.presentation`
`.inline`
`.compact` - kinda carousel-like.

## demo
Use `.ignoresSafeArea()` modifier on picker because it handles this itself.
Use `.continuous`
`disabledCapabilities` modifier to disable unneeded elements.

embedded picker can zoom!!

* iOS
* iPadOS
* macOS
same API available to UIKit/AppKit/SwiftUI.

Just use PHPickerConfiguration.


# Options menu
* PhotosPicker and `Transferable` -> don't need to do any work to get the options menu
* PHPickerViewcontoller and NSItemProvider -> same
* Not supported for UIImagePickerController
* or any picker API with library access


# HDR and Cinematic
Avoid transcoding
* Use .current encoding policy
* Use .image or .movie content type

[[Support HDR images in your app]]

cinvematic mode video
* picker gives you the rendered version
* request library access and use PhotoKit to get decision points
[[Support Cinematic mode videos in your app]]

# Wrap up
* embedded picker brings flexibility
* Options menu gives users more control
* Avoid library access unless necessary


# Resources
* https://developer.apple.com/documentation/photokit/implementing_an_inline_photos_picker
* https://developer.apple.com/documentation/photokit/selecting_photos_and_videos_in_ios
