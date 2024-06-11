Learn how to adopt spatial photos and videos in your apps. Explore the different types of stereoscopic media and find out how to capture spatial videos in your iOS app on iPhone 15 Pro. Discover the various ways to detect and present spatial media, including the new QuickLook Preview Application API in visionOS. And take a deep dive into the metadata and stereo concepts that make a photo or video spatial.

###  Spatial video capture on iPhone 15 Pro - 6:19
```swift
class CaptureManager {
    var session: AVCaptureSession!
    var input: AVCaptureDeviceInput!
    var output: AVCaptureMovieFileOutput!
    
    func setupSession() throws -> Bool {
        session = AVCaptureSession()
        session.beginConfiguration()
        
        guard let videoDevice = AVCaptureDevice.default(.builtInDualWideCamera, for: .video, position: .back) else {
            return false
        }
        
        var foundSpatialFormat = false
        
        for format in videoDevice.formats {
            if format.isSpatialVideoCaptureSupported {
                try videoDevice.lockForConfiguration()
                videoDevice.activeFormat = format
                videoDevice.unlockForConfiguration()
                foundSpatialFormat = true
                break
            }
        }
        
        guard foundSpatialFormat else {
            return false
        }
        
        let videoDeviceInput = try AVCaptureDeviceInput(device: videoDevice)
        
        guard session.canAddInput(videoDeviceInput) else {
            return false
        }
        
        session.addInput(videoDeviceInput)
        input = videoDeviceInput
        
        let movieFileOutput = AVCaptureMovieFileOutput()
        
        guard session.canAddOutput(movieFileOutput) else {
            return false
        }
        
        session.addOutput(movieFileOutput)
        output = movieFileOutput
        
        guard let connection = output.connection(with: .video) else {
            return false
        }
        
        guard connection.isVideoStabilizationSupported else {
            return false
        }
        
        connection.preferredVideoStabilizationMode = .cinematicExtendedEnhanced
        
        guard movieFileOutput.isSpatialVideoCaptureSupported else {
            return false
        }
        
        movieFileOutput.isSpatialVideoCaptureEnabled = true
        
        session.commitConfiguration()
        session.startRunning()
        
        return true
    }
}
```

###  Observing spatial capture discomfort reasons - 9:13
```swift
let observation = videoDevice.observe(\.spatialCaptureDiscomfortReasons) { (device, change) in
    guard let newValue = change.newValue else { return }
    
    if newValue.contains(.subjectTooClose) {
        // Guide user to move back
    }
    
    if newValue.contains(.notEnoughLight) {
        // Guide user to find a brighter environment
    }
}
```

###  PhotosPicker - 9:58
```swift
import SwiftUI
import PhotosUI

struct PickerView: View {
    @State var selectedItem: PhotosPickerItem?
    
    var body: some View {
        PhotosPicker(selection: $selectedItem, matching: .spatialMedia) {
            Text("Choose a spatial photo or video")
        }
    }
}
```

###  PhotoKit - all spatial assets - 10:14
```swift
import Photos

func fetchSpatialAssets() {
    let fetchOptions = PHFetchOptions()
    fetchOptions.predicate = NSPredicate(format: "(mediaSubtypes & %d) != 0", argumentArray: [PHAssetMediaSubtype.spatialMedia.rawValue])
    fetchResult = PHAsset.fetchAssets(with: fetchOptions)
}
```

###  AVAssetPlaybackAssistant - 10:36
```swift
import AVFoundation

extension AVURLAsset {
    func isSpatialVideo() async -> Bool {
        let assistant = AVAssetPlaybackAssistant(asset: self)
        let options = await assistant.playbackConfigurationOptions
        return options.contains(.spatialVideo)
    }
}
```
# Resources
https://developer.apple.com/library/content/samplecode/AVCam
https://developer.apple.com/documentation/avfoundation/media_reading_and_writing/converting_side-by-side_3d_video_to_multiview_hevc_and_spatial_video
https://developer.apple.com/documentation/ImageIO/Creating-spatial-photos-and-videos-with-spatial-metadata
https://developer.apple.com/documentation/ImageIO/writing-spatial-photos
