#vision 

# What is a data scanner?
Typicalyl data comes in the form of text.  ex, telephone numbers, dates, prices, etc.

Maybe QR codes.  Camera app, live text, etc.  Custom scanning experiences.

# How would you build one?
iOS SDK has more than one solution.
1.  AVFoundation framework to set up the camera graph.  Connect inputs/outputs, etc.
2. AVFoundation and visiom frameworks together.  Results in the delivery of CMSampleBufferRef, that can be fed to vision framework.  Vision observatin objects.
See [[Extract document data using Vision]]

New option that encapsulates all that for you.  `DataScannerViewController` in #vision kit.

# User features
A live camera preview
guidance
item highlighting
tap to focus
pinch to zoom

# Developer features
* UIViewController subclass
* View coordinates
* Region of interest
* Text content types
* Multiple symbologies

With DSVC, our goal is to perform the common tasks for you.  

Privacy usage description.  When apps try to capture video, grant explicit permission to access camera.  Provide a descriptive messsage justifying your need.  Add a `privacy - camera usage description`.  Be as descriptive asp ossible so users know what they're agreeing to.

check `.isSupported`, as we don't support it on all devices.

2018 or newer iPHone/iPad with Neural Engine supports.
Check `.isAvailable`.  If the user approves the app for camera access, the device is free of restrictions, etc.

```swift
import VisionKit

// Specify the types of data to recognize
let recognizedDataTypes:Set<DataScannerViewController.RecognizedDataType> = [
    .barcode(symbologies: [.qr]),
  	// uncomment to filter on specific languages (e.g., Japanese)
    // .text(languages: ["ja"])
    // uncomment to filter on specific content types (e.g., URLs)
		// .text(textContentType: .URL)
]

// Create the data scanner, present it, and start scanning!
let dataScanner = DataScannerViewController(recognizedDataTypes: recognizedDataTypes)
present(dataScanner, animated: true) {
    try? dataScanner.startScanning()
}
```

Pass a list of languages with a text recognizer ahs a hint for various processing.  If you know what languages to expect, list them out.  It's useful when two languages have similarly-looking scripts.  User's preferred language used by default.

```swift
// Specify the types of data to recognize
let recognizedDataTypes:Set<DataScannerViewController.RecognizedDataType> = [
    .barcode(symbologies: [.qr]),
    .text(textContentType: .URL)
]

// Create the data scanner, present it, and start scanning!
let dataScanner = DataScannerViewController(recognizedDataTypes: recognizedDataTypes)
dataScanner.delegate = self.present(dataScanner, animated: true) {
    try? dataScanner.startScanning()
}
```

Let's talk about other options.
barcode symbologies => all the ones invision.
language => same as live text.  Adding support for new japanes and korean.
Use `.supportedTextREcognitionLanguages` for the up to date list as this may change.

Seven types of data.

Let's go over initialization parameters.  therea re others taht can help.

# Parameters
| parameter                      | description                                                             |
| ------------------------------ | ----------------------------------------------------------------------- |
| recognizedDataTypes            | Text and/or codes and what types of eahc                                |
| qualityLevel                   | balanced, fast, or accurate.  Consider how readable your input will be. |
| recognizesMultipleItems        | Find one item at a time or several.  When false, centermost by default  |
| isHighFrameRateTrackingEnabled | Enable when you want hilights to follow items as closely as possible    |
| isPinchToZooomEnabled          | duh                                                                     |
| isGuidanceEnabled              | Allow for labels to help the user                                       |
| isHighlightingEnabled          | Disable for now highlighting or to draw your own                                                                        |


Provide a delegate.  Can now implement,
```swift
func dataScanner(_ dataScanner: DataScannerViewController, didTapOn item: RecognizedItem) {
    switch item {
    case .text(let text):
        print("text: \(text.transcript)")
    case .barcode(let barcode):
        print("barcode: \(barcode.payloadStringValue ?? "unknown")")
    default:
        print("unexpected item")
    }
}
```

receive an instance of RecognizeItem.  enum that holds text or a barcode as an associated value.  For text, the transcription property holds the recognized string.  For barcodes, reetrieve it with the payloadString value.

## RecognizedItem
* unique identifier
* stays with object as longa s its seen
* Bounds (four corners, not a rect)

## Delegate methods

```swift
// Dictionary to store our custom highlights keyed by their associated item ID.
var itemHighlightViews: [RecognizedItem.ID: HighlightView] = [:]

// For each new item, create a new highlight view and add it to the view hierarchy.
func dataScanner(_ dataScanner: DataScannerViewController, didAdd addItems: [RecognizedItem], allItems: [RecognizedItem]) {
    for item in addedItems {
        let newView = newHighlightView(forItem: item)
        itemHighlightViews[item.id] = newView
        dataScanner.overlayContainerView.addSubview(newView)
    }
}
```
Add items to the `overlayContainerView` so they have the right z-order.
```swift
// Animate highlight views to their new bounds
func dataScanner(_ dataScanner: DataScannerViewController, didUpdate updatedItems: [RecognizedItem], allItems: [RecognizedItem]) {
    for item in updatedItems {
        if let view = itemHighlightViews[item.id] {
            animate(view: view, toNewBounds: item.bounds)
        }
    }
}
```
Can be called when transcription changes.  Longer we have, the more accurate it is.


```swift
// Remove highlights when their associated items are removed.
func dataScanner(_ dataScanner: DataScannerViewController, didRemove removedItems: [RecognizedItem], allItems: [RecognizedItem]) {
    for item in removedItems {
        if let view = itemHighlightViews[item.id] {
            itemHighlightViews.removeValue(forKey: item.id)
            view.removeFromSuperview()
        }
    }
}
```

Forget about any highlight views with the removed items.  Remove from view hierarchy.  IF you draw your own highlights over items, these will be crucial for animating higlights.  

And you get an arrya of items currently recognized.  Items are placed in natural reading order.  

# Feature grab bag
## Photo capture
```swift
// Take a still photo and save to the camera roll
if let image = try? await dataScanner.capturePhoto() {
    UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil)
}
```

## `recognizedItems` async stream
```swift
// Send a notification when the recognized items change.
var currentItems: [RecognizedItem] = []

func updateViaAsyncStream() async {
    guard let scanner = dataScannerViewController else { return }
    
    let stream = scanner.recognizedItems
    for await newItems: [RecognizedItem] in stream {
        let diff = newItems.difference(from: currentItems) { a, b in
            return a.id == b.id
        }
        
        if !diff.isEmpty {
            currentItems = newItems
            sendDidChangeNotification()
        }
    }
}
```

# Wrap up
AVFoundation and vision
`DataScannerViewController` in VisionKit
Custom experience
[[Add live text interaction to your app]]

* https://developer.apple.com/forums/tags/wwdc2022-10025
* https://developer.apple.com/forums/create/question?&tag1=255&tag2=471030
* https://developer.apple.com/documentation/visionkit/scanning_data_with_the_camera