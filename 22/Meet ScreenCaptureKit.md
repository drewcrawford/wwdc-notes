#screencapturekit

On top of that, streaming gameplay has been a continually growing area for people's education and entertainment.  With this in mind, we created a framework for performant and robust screencapture.

* brand new fraemwork on macOS designed to create screen sharing experience
* Custom content capture
* Developer controls
* Dynamic updates
* high quality and performance up to native resolution and framerate of your display
* global privacy safeguards

[[Take ScreenCaptureKit to the next level]]

# Key features
* supports diplay, application, and window capture
* corresponding audio capture

developer controls
* pixel format, color space, frame rate, resolution
* sample rate, channel count

dynamic updates
* adjusted and configuration dynamically
high quality and performance
* up to 48khz stereo audio
* up to native resolution and frame rate
* GPU utilization with lower CPU overhead

privacy safeguards
* requires permission to capture screen content
* Persistent setting stored in system preferences

# API overview
* SCStream
	* control methods like start/stop
* SCShareableContent
* SCContentFilter
* SCStreamConfiguration
* SCStreamOutput

# Stream setup
what you capture, quality/performance.

## SCShareableContent
On this desktop there are windows, applications, and the display itself.  SCK has a corresponding class for each.

SCDisplay properties
* identifier
* sidth
* height
* SCRunningApplications

SCRunningApplication
* bundle identifier
* application name
* process identifier
* windows

SCWindow properties
* window identifier
* frame
* title
* on screen (vs minimized)
* owning application

all 3 are SCShareableContent.  Get a list of all shareable content on the device, or you can specify certain parameters.

ex, list windows onscreen.  Simple API.
```swift
// Creating a SCShareableContent object

// Get the content that's available to capture.
let content = try await SCShareableContent.excludingDesktopWindows(
    false,
    onScreenWindowsOnly: true
)
```

## SCContentFilter
* display-independent window filter
	* captuers the window as it moves across displays.
* Display-dependent with applications or windows filtered
	* exclude your own application?
* audio filtered per application only

```swift
// Creating a SCContentFilter object

// Get the content that's available to capture.
let content = try await SCShareableContent.excludingDesktopWindows(
    false,
    onScreenWindowsOnly: true
)

// Exclude the sample app by matching the bundle identifier.
let excludedApps = content.applications.filter { app in
    Bundle.main.bundleIdentifier == app.bundleIdentifier
}

// Create a content filter that excludes the sample app.
filter = SCContentFilter(display: display,
                         excludingApplications: excludedApps,
                         exceptingWindows: [])
```

## Capture configuration
these controls can be set in SCStreamConfiguration.

* output resolution
* framerate
* whether or not to show the mouse cursor

audio side: 
* enable
* change sample rate
* change channel count

ex:
* low motion, text clarity.  Notes, spreadsheet
	* 4k, 10fps
	* leave audio disabled
* high motion
	* prioritize framerate over resolution
	* output resolution 1080p
	* fps=60
	* hide cursor movement

audio capture enabled for more immersive experience.

```swift
// Creating a SCStreamConfiguration object
let streamConfig = SCStreamConfiguration()
        
// Set output resolution to 1080p
streamConfig.width = 1920
streamConfig.height = 1080

// Set the capture interval at 60 fps
streamConfig.minimumFrameInterval = CMTime(value: 1, timescale: CMTimeScale(60))

// Hides cursor
streamConfig.showsCursor = false

// Enable audio capture
streamConfig.capturesAudio = true

// Set sample rate to 48000 kHz stereo
streamConfig.sampleRate = 48000
streamConfig.channelCount = 2
```

Pass in an optional delegate in order to handle errors.  STart capture, and sck will provide stream with samples when available.  With filter and configuration created, startin ga stream is easy.

```swift
// Creating and starting a SCStream object

// Create a capture stream with the filter and stream configuration
stream = SCStream(filter: filter, configuration: streamConfig, delegate: self)

// Start the capture session
try await stream?.startCapture()      


// ...
// Error handling delegate 
func stream(_ stream: SCStream, didStopWithError error: Error) {
    DispatchQueue.main.async {
        self.logger.error("Stream stopped with error: \(error.localizedDescription)")
        self.error = error
        self.isRecording = false
   }
}
```



# Getting media samples

sent in the form of CMSample objects.  In order to get media samples, need to add stream output.

When you add stream output, you may specify handler queue.  may be useful if you want sample to be delivered in a particular queue.  If you don't specify, a default queue will be used.

CMSampleBuffer with SCStreamOutputType delivered to your application when available.

```swift
// SCStreamOutput protocol implementation
func stream(_ stream: SCStream, didOutputSampleBuffer sampleBuffer: CMSampleBuffer, of type: SCStreamOutputType) {
    switch type {
    case .screen:
        handleLatestScreenSample(sampleBuffer)
    case .audio:         handleLatestAudioSample(sampleBuffer)
    }
}

// ...
// Create a capture stream with the filter and stream configuration
stream = SCStream(filter: filter, configuration: streamConfig, delegate: self)

// Add a stream output to capture screen and audio content
try stream?.addStreamOutput(self, type: .screen, sampleHandlerQueue: screenFrameOutputQueue)
try stream?.addStreamOutput(self, type: .audio, sampleHandlerQueue: audioFrameOutputQueue)

// Start the capture session
try await stream?.startCapture()
```

delivers samples in CMSampleBuffers.

On video side, it's IO-surface-backed.  
Per sample attachment, SCStreamFrameInfo.  Check for frame status.
SCStreamFrameSTatus - complete, idle
Existing CMSampleBuffer utilities available.

# Wrap up
* filtered content cpature
* Customizable configuration
* Getting started with ScreenCaptureKit

[[Take ScreenCaptureKit to the next level]]
CGDisplayStream and CGWindowList will be deprecated in the future.


https://developer.apple.com/documentation/screencapturekit
https://developer.apple.com/documentation/screencapturekit/capturing_screen_content_in_macos
