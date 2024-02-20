Level up your screen sharing experience with the latest features in ScreenCaptureKit. Explore the built-in system picker, Presenter Overlay, and screenshot capabilities, and learn how to incorporate these features into your existing ScreenCaptureKit app or game.
###  3:32 - Set up delegate for stream
```swift
// Set up delegate for stream
let stream = SCStream(filter: filter, configuration: config, delegate: self)

// delegate method for Presenter Overlay applied
func stream(_ stream: SCStream, outputEffectDidStart didStart: bool) {
    // if Presenter Overlay is on, present banner in app to notify
    if didStart == true {
        presentBanner()
        turnOffCamera()
    } else {
        turnOnCamera()
    }
}
```

###  6:48 - Set up content sharing picker instance
```swift
// Set up content sharing picker instance
    let picker = SCContentSharingPicker.shared()
    picker.addObserver(self)
    picker.active = true
    
    // show system level picker button
    func showSystemPicker(sender: UIButton!) {
        picker.present(for stream: nil, using contentStyle:.window)
    }
    
    // observer call back for picker
    func contentSharingPicker(_ picker: SCContentSharingPicker, didUpdateWith filter:                                          
    SCContentFilter, for stream: SCStream?) {
       if let stream = stream {
            stream.updateContentFilter(filter)
        } else {
            let stream = SCStream(filter: filter, configuration: config, delegate: self)
        }
    }
```

###  7:41 - Observer call back for picker did fail and did cancel
```swift
// Set up content sharing picker instance
    let picker = SCContentSharingPicker.shared()
    picker.addObserver(self)
    picker.active = true

    // show system level picker button
    func showSystemPicker(sender: UIButton!) {
        picker.present(for stream: nil, using contentStyle:.window)
    }

    // observer call back for picker did fail
    func contentSharingPicker(contentSharingPickerStartDidFailWith error:NSError) {
        if error {
            presentNotifications(error: error)
        }
    }

    // observer call back for picker did cancel
    func contentSharingPicker(_ picker: SCContentSharingPicker, didCancel for stream: SCStream?) {
       if stream {
           resetStateForStream(stream: stream)
       }
    }
```

###  8:41 - Per-stream configuration
```swift
// Set up content sharing picker instance
    let picker = SCContentSharingPicker.shared()
    picker.addObserver(self)
    picker.active = true
    
    // Create configurations
    let pickerConfig = SCContentSharingPickerConfiguration()
    
    // Set Picker configuration
    pickerConfig.excludedBundleIDs = [“com.foo.myApp”,”com.foo.myApp2”]
    pickerConfig.allowsRepicking = true
    
    // Create configurations
    picker.setConfiguration(pickerConfig, for: stream)

    func showSystemPicker(sender: UIButton!) {
        picker.present(for stream: nil, using contentStyle:.window)
    }
```

###  12:26 - Call the screenshot API
```swift
// Call the screenshot API

class SCScreenshotManager : NSObject {

class func captureSampleBuffer(contentFilter: SCContentFilter, 
                               configuration: SCStreamConfiguration)
  															async throws -> CMSampleBuffer

class func captureImage(contentFilter: SCContentFilter,
                        configuration: SCStreamConfiguration)
  											async throws -> GImage
}
```

###  12:44 - Take a screenshot with ScreenCaptureKit
```swift
// Don't forget to customize the content you want in your screenshot
// Use SCShareableContent or SCContentSharingPicker to pick your content
let display = nil;

// Create your SCContentFilter and SCStreamConfiguration
// Customize these lines to use the content you want and desired config options
let myContentFilter = SCContentFilter(display: display,
                             excludingApplications: [],
                             exceptingWindows: []);
let myConfiguration = SCStreamConfiguration();

// Call the screenshot API and get your screenshot image
if let screenshot = try? await SCScreenshotManager.captureSampleBuffer(contentFilter: myContentFilter, configuration:
                                                       myConfiguration) {
    print("Fetched screenshot.")
} else {
    print("Failed to fetch screenshot.")
}
```
# Resources
* https://developer.apple.com/documentation/screencapturekit
