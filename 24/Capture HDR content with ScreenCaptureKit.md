
Learn how to capture high dynamic colors using ScreenCaptureKit, and explore new features like HDR support, microphone capture, and straight-to-file recording.

### Capture HDR - 5:22
```swift
// Get content that is currently available for capture.
let availableContent = try await SCShareableContent.current
        
// Create instance of SCContentFilter to record entire display.
guard let display = availableContent.displays.first else { return }
let filter = SCContentFilter(display: display, excludingWindows: [])

// Create a configuration using preset for HDR stream canonical display.
let config = SCStreamConfiguration(preset: .captureHDRStreamCanonicalDisplay)

// Create a stream with the filter and stream configuration.
let stream = SCStream(filter: filter, configuration: config, delegate: self)

// Add a stream output to capture screen content.
try stream.addStreamOutput(self, type: .screen, sampleHandlerQueue: nil)

// Start the capture session.
try await stream.startCapture()
```

### Use a local display preset to capture HDR - 6:40
```swift
// Create an SCStreamConfiguration with preset for HDR.
let config = SCStreamConfiguration(preset: .captureHDRScreenshotLocalDisplay)

// Call the screenshot API to get CMSampleBuffer representation
let screenshotBuffer = try await SCScreenshotManager.captureSampleBuffer(contentFilter: filter, configuration: config)

// Call the screenshot API to get CGImage representation.
let screenshotImage = try await SCScreenshotManager.captureImage(contentFilter: filter, configuration: config)
```

### Capture samples of microphone audio - 8:05
```swift
// Create instance of SCStreamConfiguration.
let config = SCStreamConfiguration()

// Enable microphone capture and set id of microphone to capture.
config.captureMicrophone = true
config.microphoneCaptureDeviceID = AVCaptureDevice.default(for: .audio)?.uniqueID

// Create an SCStream instance.
let stream = SCStream(filter: filter, configuration: config, delegate: self)

// Add stream outputs for capturing screen and microphone.
try stream.addStreamOutput(self, type: .screen, sampleHandlerQueue: nil)
try stream.addStreamOutput(self, type: .microphone, sampleHandlerQueue: nil)

// Start the capture session
try await stream.startCapture()

// Implement SCStreamOutput function to receive samples.
func stream(_ stream: SCStream, didOutputSampleBuffer sampleBuffer: CMSampleBuffer, of type: SCStreamOutputType) {
    switch type {
    case .screen:
        handleLatestScreenSample(sampleBuffer)
    case .audio:         
        handleLatestAudioSample(sampleBuffer)
    case .microphone:
        handleLatestMicrophoneSample(sampleBuffer)
    }
}
```

### Record to file - 9:38
```swift
// Create a recording output configuration.
let recordingConfiguration = SCRecordingOutputConfiguration()

// Configure the outputURL (optionally set file type and video codec).
recordingConfiguration.outputURL = url
recordingConfiguration.outputFileType = .mov
recordingConfiguration.videoCodecType = .hevc
        
// Create the recording output with the configuration.
let recordingOutput = SCRecordingOutput(configuration: recordingConfiguration, delegate: self)

// Add an SCRecordingOutput to the stream.
try stream.addRecordingOutput(recordingOutput)

// Start capturing (which will also start recording).
try await stream.startCapture()

// Stop recording.
try await stream.stopCapture()

//OR
// Stop recording, but keep stream running.
try stream.removeRecordingOutput(recordingOutput)
```

### Respond to delegate events - 10:27
```swift
func recordingOutputDidStartRecording(_ recordingOutput: SCRecordingOutput) {
    // Recording started asynchronously after addRecordOutput.
}

func recordingOutput(_ recordingOutput: SCRecordingOutput, didFailWithError error: Error) {
    // Recording failed with error.
}
        
func recordingOutputDidFinishRecording(_ recordingOutput: SCRecordingOutput) {
    // Recording finished after calling either removeRecordOutput or stopCapture.
}
```

# Resources
* [Forum: Media Technologies](https://developer.apple.com/forums/topics/media-technologies?cid=vf-a-0010)
* [ScreenCaptureKit](https://developer.apple.com/documentation/screencapturekit)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10088/4/D333573B-E8F2-4420-8709-B8FE3095D56B/downloads/wwdc2024-10088_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10088/4/D333573B-E8F2-4420-8709-B8FE3095D56B/downloads/wwdc2024-10088_sd.mp4?dl=1)