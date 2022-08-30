# 4k video
Capture.  Binning.

2x2 pixels, averages the piuxel values, and writes back a single pixel.

First, image dimensions are downscaled by a factor of 2.  Each frame consumes less memory and processing power.  Run the camera at up to 60fps, etc.

This process offers an advantage in low-light environments.  

RealityKit renders and composits the virtual content on top of the frame to display the final result on screen.  Now we enable 4k video.

You can skip the binning step and directly access in 4k resolution.  3840x2160.  30fps.  Apart from these changes, you work the same as before.

REalityKit performs scaling, cropping, rendering.

#realitykit 



iPhone 11 and up, latest iPad pro (M1)
3840x2160px
30fps
16:9 aspect ratio

## Best practices
* Hold onto ARFrame only as long as eneded
* Might stop ARKit from surfacing new ARFrames
* trackingState might be limited
* check for console warnings
* Only use 4k when there is a clear need
* We still suggest using full HD at 60fps for games and stuff.
# Camera enhancements
* high resolution background photos
* HDR
* AVCaptureDevice access
* Exif tags

## high resolution background photos
full 12mp or whatever.

helping people position a photo.
object capture.

[[Bring your world into augmented reality]]

```swift
if let hiResCaptureVideoFormat = ARWorldTrackingConfiguration.recommendedVideoFormatForHighResolutionFrameCapturing {
    // Assign the video format that supports hi-res capturing.
config.videoFormat = hiResCaptureVideoFormat
}
// Run the session.
session.run(config)
```

calling this function triggers an out-of-band capture of highres image.

```swift
session.captureHighResolutionFrame { (frame, error) in
   if let frame = frame {
      saveHiResImage(frame.capturedImage)
   }
}
```

## HDR
```swift
if (config.videoFormat.isVideoHDRSupported) {
    config.videoHDRAllowed = true
}
session.run(config)
```

(currently only non-binned formats)

This will have a performance impact, only use when you need it.

## AVCaptureSession
```swift
if let device = ARWorldTrackingConfiguration.configurableCaptureDeviceForPrimaryCamera {
   do {
      try device.lockForConfiguration()
      // configure AVCaptureDevice settings
      …
      device.unlockForConfiguration()
   } catch {
      // error handling
      …
   }
}
```

Get access toAVCaptureDevice for fine-grained control.  Keep in mind this is used by ARKit, so any changes might have negative effect of the output quality of ARKit.

See AVCapture docs.

## EXIF tags
on every ARFrame.  Whtie balance, exposure, etc.  

# Plane anchors
Typical plane anchor in iOS 15.  Fits plane to book on the table.  Plane extends etc.

Plane anchor and geometry updates are now fully decoupled.  Now the anchor rotation itself remains constant.

* ARPlaneExtent
* rotationOnYAxis
* width
* height
* ARPlaneAnchor.center

```swift
// Create a model entity sized to the plane's extent.
let planeEntity = ModelEntity(
    mesh: .generatePlane (
        width: planeExtent.width, 
        depth: planeExtent.height),
    materials: [material])

// Orient the entity.
planeEntity.transform = Transform(
    pitch: 0, 
    yaw: planeExtent.rotationOnYAxis, 
    roll: 0)

// Center the entity on the plane.
planeEntity.transform.translation = planeAnchor.center
```


# Motion capture
2 new joints: left and right ear (2d).
Improved pose detection (2d)
more temporal consistency (3d)
Better occlusion handling (3d)
set DT to iOS 16.
# Location anchors
Apple maps uses the location anchor API to power pedestrian walking instructions.  

* vancouver, toronto, montreal
* singapore
* various japanese cities
* melbourne, sydney
later this year:
* auckland nz
* tel aviv
* paris

# Wrap up
* 4k video
* camera enhancements
* plane anchors
* motion capture
* location anchors

 

* https://developer.apple.com/forums/tags/wwdc2022-10126
* https://developer.apple.com/forums/create/question?&tag1=34&tag3=441030
* https://developer.apple.com/design/human-interface-guidelines/ios/system-capabilities/augmented-reality
* https://developer.apple.com/forums/tags/arkit
* https://developer.apple.com/documentation/arkit/content_anchors/tracking_geographic_locations_in_ar
* https://developer.apple.com/documentation/arkit
