#replaykit

In-app screen capture for additional filters or overlays.

Audio/video samples directly to your application process.

In-app screen broadcast.  Stream to 3rd party broadcast services.

Players would need to record entire gameplay.  Large recording, require time to trim, etc.

Why doesn't replaykit do this for you?

# In-app clips recording
Rolling buffer of samples
Samples up to 15sec prior can be exported.

Several ways to export

* User-driven clips
* Achievement driven clips

Create personalized user experiences.  Custom overlay that opens to a collection recording.  

Available for iOS, macOS.  Sits near existing recording, capture, broadcast APIs.

* HD quality
* Low performance impact
* Privacy safeguards

3 APIs.

* start
* stop
* export

`RPScreenRecorder`shared instance.

After the rolling buffer starts, RK waits for your app to export.  App calls exportClip API.

Sample project.

```swift
// Start clip buffering API call

func startClipBuffering() {
    RPScreenRecorder.shared().startClipBuffering { error in
        if error != nil {
            print("Error attempting to start Clip Buffering")
            // Update the app recording state and UI.
            self.setClipState(active: false)
        } else {
            // No error encountered attempting to start a clip session.
            // Update the app recording state and UI.
            self.setClipState(active: true)
            
            // Set up camera View.
            self.setupCameraView()
        }
    }
}
```

```swift
// Stop clip buffering

func stopClipBuffering() {
    RPScreenRecorder.shared().stopClipBuffering { error in
        if error != nil {
            print("Error attempting to stop clip buffering")
        }
        // Update the app recording state and UI.
        self.setClipState(active: false)
        
        // Tear down camera view.
        self.tearDownCameraView()
    }
}
```

Exporting

```swift
// Export clip button

@IBAction func exportClipButtonTapped(_ sender: Any) {
    // If clip buffering is active, export clip
    if self.isActive && self.getClipButton.isEnabled {
        exportClip()
    }
}
```

You may export clips automatically if you desire.

```swift
// Export clip

func exportClip() {
    let clipURL = getAppTempDirectory()
    let interval = TimeInterval(5)

    print("Generating clip at URL: \(clipURL)")
    RPScreenRecorder.shared().exportClip(to: clipURL, duration: interval) { error in
        if error != nil {
            print("Error attempting to export clip")
        } else {
            // No error, so save clip at URL to photos
            self.saveToPhotos(tempURL: clipURL)
        }
    }
}
```

Have the clip at the specified URL.  With access to the clips, you can build and organize your own UX.

Clip is simply saved to photos.

Direct access to clips in the specified URL.  Build your own in-app experiences.

## Game controller integration
* start/stop recording from controller
* Recordings automatically saved to Photos or Desktop.

* Key value observing `available`
* Key value observing `recording`
* RPScreenRecorderDelegate protcol

# Wrap up
* Simple to integrate
* Capture all exciting moments, when they happen
https://developer.apple.com/documentation/replaykit/recording_and_streaming_your_macos_app
https://developer.apple.com/documentation/replaykit


* 