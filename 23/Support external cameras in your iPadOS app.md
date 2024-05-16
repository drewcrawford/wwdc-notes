Learn how you can discover and connect to external cameras in your iPadOS app using the AVFoundation capture classes. We'll show you how to rotate video from both external and built-in cameras, support external microphones with USB-C, and perform audio routing. Explore telephony support, tunings for optimal echo cancellation, and best practices for external camera adoption.

Stage manager includes iPad display across multiple screens.  Now we can start using external cameras like from apple studio display.

Ipads with usb-c connectors
conform to USB video class (UVC) specification
Many popular cameras
Use included microphones
non-camera devices like HDMI switchers.

# Discovery and usage
AVFoundation allows your app to use built-in and external cameras.  Speicfically, AVCapture-prefixed classes.

First we use AVCaptureDevice camera/mic.
Wrapped in AVCaptureDeviceInput.
AVCaptureSession, central control object
AVCaptureOutputs -> render data from inputs in various ways.

Live camera preview -> aVCaptureVideoPreviewLayer.

data flows from inputs to session to outputs.  iOS, macOS, tvOS.

Your app can access external cameras with AVCapture.
Simple updates to start using
Represented by AVCaptureDevice instances.
* avcapturedevice
* AVCaptureDevice.DiscoverySession

3 main attributes
* media type -> video
* device type -> external
* position -> unspecified

Easy to start using external cameras in your app. In this session, I'll modify AVCam to stream from external camera.  See sample code.

## connection/disconnection events

User can connect/disconnect at any time
Monitor events for camera availability
If reconnected, camera is represented by new AVCaptureDevice
Use existing API for listening to events

KVO -> AVCaptureDevice.isConnected
or AVCAptureDevice.DiscoverySession.devices

Listen for notifications
AVCaptureDeviceWasConnected, etc.

KVO is called from background queues.  Synchronize with AVCaptureSession and UI.

## automatic camera selection
IPadOS is introducing api for automatic camera selection.
Integrate with system
Another way to change cameras.

[[Bring Contintuity Camera to your macOS app]]

use two-new class properties on AVCaptureDevice: `userPreferredCamera` and `systemPreferredCamera`.  Both KVO-observable

`userPreferredCamera` -> user's choice.  Set whenever a user picks a camera.  Doing so alows the system to learn the user's preference.

`systemPreferredCamera` -> readonly, as determined by the system.  By default, we use front camera.  Can inform the system to use back camera instead.  Influenced by user choice.  

How does the system know which camera is best?

System stores a short history of chosen cameras. Across launches and reboots.

`.userPreferredCamera`.  If a camera is disconnected, system returns next available camera based on user's history.  If no user selection history, system will always try to return a camera that's ready to use, prioritizing prior use.

Store your user's camera preference!  

API is flexible for desired behavior.  
Support on iPhone/iPad.
Automatic and manual camera selection.
FaceTime, Code Scanner, and WebKit are great examples of camera selection behaviors.

When FT launches, uses front or external camera.  Can switch between built-in cameras on calls.
No switching when external camera is used.

code scanner
* prefer back camera

webkit example.
* allows switching to any camera
* returns system preferred camera first


I"ll choose to treat an external camera like front-facing.

# Demystifying video rotation

The app doesn't know how to orient external cameras.  AVCam will need further modifications to properly display video preview.

* not a new concept for apps
* external cameras do not rotate with the iPad
* Apps are used to built-in cameras
* tend to rely on iPad orientation
	* AVCaptureVideoOrientation

This is now deprecated!  Describes how iPad is oriented, assumes camera rotates.
Not expressive enough for external cameras which rotate independently.

We're introducing new API to handle video rotation.  New to iPadOS,

AVCaptureDevice.RotationCoordinator

device, and CALayer.  Pass the layer of your preview view.

These return an angle in degrees and are KVO-observable.  

Use to display video frames in the CALayer.  How much rotation for preview.  Angle relative to UIKit/SwiftUI.

Capture upright photos and movies
Physical orientation of camera
Different from angle needed for preview.

preview vs capture.  Origin in top left.  Camera sensor origin is top right.  App rotates video frames 90 degrees.  

In landscape.  Camera app on iPhone is always portrait.  UIkit coordinate is still in top left (of portrait orientation).  Camera coordinate system still differs from UI.  It applies a constant 90 degrees of rotation no matter iphone orientation.

But when in native orientation, we don't need to rotate photos/movies for them to appear upright.

Don't ask questions, trust the coordinator.
Rely on it rather than calculating angles.
Use a new coordinator when switching cameras!

Use preview
* for displaying camera preview
* with AVCaptureVideoPreviewLayer
* or dispalyign buffer from AVCapturevideoDataoutput in a CALayer
Synchronize with system animations
* apply rotation directly in KVO handler
* Delivers updates on main queue

Rotating video buffers for preview

Displaying video buffers
* from AVCaptureVideoDAtaOutput
* for custom effects or filters
* i.e. in a AVSampleBufferDisplayLayer
* AVoid rotating with aVCaptureConnection
* * interruption of frame delivery!
* Instead rotate CALayer displaying the camera preview.  Allows smooth rotation.

Use capture property for capturing photos/movies.  
Set AVCaptureconnection.videoRotationAngle, for PhotoOutput, MovieOutput
When recording AVASsetWriter, write unrotated buffers and then set AVAssetWriterInput.transform, this alters the metadata.  This uses less energy than rotating each frame.
Convert from degrees to radians, because assetWriterInput uses CGAffineTransform in radians.

Efficient rotation.  
* movie output -> quicktime metadata
* AVCapturePhotoOutput -> EXIF tags
* previewlayer output -> transforms

Expensive rotation:
* AVCaptureVideoDAtaOutput
* AVCaptureDepthDataOutput
* we recommend rotating CALayer for best preview performance.

Use AVCaptureDevice.RotationCoordinator on all platforms.
Capture and preview for any camera
Correct video rotation with compelx layouts.  Stage manager, external display, etc.

When creating a coordinator, app updates preview layer.  it also observes changes to the angle and updates the preview.

# External microphones

Webcams may include mics.  iPad, used by your app. iPadOS 17 improves support with external microphones on iPads with USB-C.
Telephony support with AUVoiceIO.
Previously we only supported headset mics.
Echo cancellation
Voice isolation mode

Only one mic can be used at a time
last one wins
* indicates user intent

On iOS, only one AVCAptureDevice returned for microphone
AVMediaType.audio.
New aVCaptureDevice.DeviceType.microphone
builtInMicrophone is deprecated

localizedName changes with active microphone

Use AVAudioSession for more control
Configure app's audio behavior
AVAudioSession.Category
AVAudioSession.Mode

Set preferredInput

# Best practices
* do what makes most sense for your app!
* Wireless Xcode debugging
* Handle cameras with different capabilities
* For example, no depth data capture
* AVCaptureMultiCamSession support

Video preview is mirrored by default
Like front-facing cameras
Mirroring may not be suitable for all usecases

External cameras rotate clockwise toward scene

| format | resolution | frame rate |
| ------ | ---------- | ---------- |
| 420v   | 640x480    | 1-30       |
| 420v   | 1280x720   | 1-30       |

Pixel formats

| native pixel format | reported format |
| ------------------- | --------------- |
| yuvs                | 420v            |
| 2vuy                | 420v            |
| jpeg/dmb1           | 420f            |
| avc1                | 420f            |

Check for `AVCaptureSession.Preset` support.  ex `.hd4K3840x2160` preset needs device format.

Can change resolution, framerate, zoom factor.  We support limited UVC controls.
Query AVCaptureDevice for capabilities.

# Wrap up
* external camera discovery and usage
* video rotation
* external microphones
* best practices

# Resources
* https://developer.apple.com/library/content/samplecode/AVCam
* https://developer.apple.com/documentation/avfoundation
* https://developer.apple.com/av-foundation/
