#watchos 

The ability to share a watch face with anyone.  Can be shared directly from the watch by long-pressing and choosing share.

# Watch face distrubtion
Watch faces can be shared from apps and websites
Watch faces include complication data
Customer will be prompted to install app
Hardware compatibility: Series 3 (not all faces available), Nike (faces lockedout), Hermes (similar)

# Watch face file
Includes
* face type
* color, styles, ...
* complications
	* sample entry
	* user data

[[Create Complications for Apple Watch]]

`addWatchFace(at:completionHandler:)`.  Evidently you call this on iOS (and watchOS?)

After this, your app runs and responds to a timeline request from clockkit.

# Example
* Generate a watch face with your app's complication
* Import watch face file and preview into project
* Display watch face preview above 'add face' button
* Add support for older watches

Your app needs to have been already published to the app store before you can share this watch face in your app.

Preview image.

Add `.watchface` to xcode project.

```swift
//detect paired watch
var isPaired: Bool {
    if (WCSession.isSupported()) {
        let session = WCSession.default
        session.delegate = self
        session.activate()
        return session.isPaired
    } else {
        return false
    }
}
```

```swift
//add face wrapper
private func addFaceWrapper(withName: String) {
    if let watchfaceURL = Bundle.main.url(forResource: withName, withExtension: "watchface") {
        CLKWatchFaceLibrary().addWatchFace(at: watchfaceURL, completionHandler: {
            (error: Error?) in
            if let nsError = error as NSError?, nsError.code == CLKWatchFaceLibrary.ErrorCode.faceNotAvailable.rawValue {
                print(nsError)
            }
            isLoading = false
        })
    }
}
```

## Supporting older watches
```swift
private func addFaceWrapper(withName: String, fallbackName: String?) {
    if let watchfaceURL = Bundle.main.url(forResource: withName, withExtension: "watchface") {
        CLKWatchFaceLibrary().addWatchFace(at: watchfaceURL, completionHandler: {
            (error: Error?) in
            if let nsError = error as NSError?, nsError.code == CLKWatchFaceLibrary.ErrorCode.faceNotAvailable.rawValue {
                if let name = fallbackName {
                    // We failed, trying with fallbackName.
                    addFaceWrapper(withName: name, fallbackName: nil)
                }
            }
            isLoading = false
        })
    }
}
```

## Best practices
* User data may be included in the watch face.  This may be valuable for specifying what content is displayed in your complication, but be careufl not to unintentionally include private data
* Provide complciation sample data to avoid preview being blank
* Detect a paired watch before offering watch face
* Get face previews via email
* Support older watches

**Your app must be published before you can include it in a shared watch face.  This is because the app store connect ID needs to be included in the shared watch face.**

# Website distribution
Safari recognizes watch face files
* customer prompted ot add face

Host the file with MIME type: `application/vnd.apple.watchface`

* Show preview of face above official button
* Support Apple Watch Series 3.  May need separate previews/buttons
* Consider sharing with QR codes and NFC tags.  Additional opportunities.  Embed a weblink that goes to `watchface` file

# Next steps
* configure watch faces with your complications
* adopt the face sharing API in your app
* Host watch faces on your website



# Resources
https://developer.apple.com/design/human-interface-guidelines/watchos/overview/themes/
https://developer.apple.com/design/resources/
https://developer.apple.com/documentation/clockkit

