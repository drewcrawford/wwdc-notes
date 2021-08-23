#camera
#avfoundation 

* AVCaptureDevice (camera)
* AVCaptureDeviceInput (wraps device) 
* AVCaptureSession
* AVCaptureOutputs

# Minimum focus distance
Distance from lens to closest point where sharp focus can be attained.

Beginning in iOS 15,

|                                  | Wide | Telephoto   |
|----------------------------------|------|-------------|
| iPhone X                         | 10cm | 40cm (2x)   |
| iPhone XS Max                    | 12cm | 40cm(2x)    |
| iPhone 12 Pro                    | 12cm | 40cm(2x)    |
| iPhone 12 Pro Max (Sensor shift) | 15cm | 50cm (2.5x) |

Can apply a zoom to prompt people to move camera away physically.  App can set this based on this property.

New AVCamBarcode sample for this property.  
# 10-bit HDR video
Multiple exposures, blending.
What about video?  It's a challenge.
EDR => HDR-like solution.  Doubles capture framerate, alternating between standard and short exposures.  When nominally capturing 30fps, camera is running at 60fps.  Not a full HDR solution, but it provides good results.

Problem: EDR was presented under the name `videoHDREnabled` etc.  But substitute EDR for this, that's what it is.

`.automaticallyAdjustsVideoHDREnabled` which defaults to true.  So EDR is enabled automatically when available.  Need to set 2 properties to false.

But 10-bit hdr video
* More bits
* Always-on EDR
* HLG BT.2020 color space
* Dolby Vision metadata
* iPhone 12 and newer

On older iPhone model, they have 420v and 420f for each resolution/framerate.  v=> video range.  f=>full range.

On iPhone 12, some formats have clusters of 3.  420v,420f, x420.  x420 is a biplanar 4,2,0 format.  But x stands for 10-bit.

`420YbCbCr10BiPlanarVideoRange`

Updated sample code.  `tenBitVariantOfFormat` which can find the 10-bit HDR variant.

This is supported for all popular formats.  

Editing is tricky.  See related talk
[[Edit and Playback HDR Video with AVFoundation]]

# Video effects in Control Center
System-level camera features
Available in your apps with no code changes
User is in control

Traditionally, 
* apple apps out of the box
* expose new capture APIs
* you learn about them
* you adopt in your app

Often results in long lead times where users miss out on great features.

With VEiCC,
* System-level camera features
* Available to your apps out of the box
* No code changes needed
* User in control
* Continue to expose new camera APIs
* Adopt at your own pace

## Center Stage
Available on M1 iPad Pro.  Enhances production value of facetime video calls.

Tracks you as you move around.  Swipe down on control center, choose video effects, make selection.  No app changes.

## Portrait
Shallow depth of field effect.  Uses neural engine plus trained monocular depth network.

## Mic modes
1.  Standard
2.  Voice isolaton
3.  Wide spectrum

## Center Stage APIs
M1 iPad Pro front cameras
UltraWide
Virtual Wide
Virtual TrueDepth (terms and conditions may apply)

* On/off state is sticky per app
* One state per app (not one state per camera)
* Settable, observable class properties on `AVCaptureDevice`
	* `AVCaptureDevice.isCenterStageEnabled`
	* `AVCaptureDevice.centerStageControlMode`
* `captureDevice.activeFormat.isCenterStageSupported`
* `captureDevice.isCenterStageActive`

Limitations

* Max frame rate of 30
* Max resolution fo 1920x1440
* Video zoom factor locked to 1
* GDC must be on
* Depth delivery must be off

### Control modes
* `.user` => default for all apps
	* Only the user may turn on/off
	* Exception thrown if you do it
* `.app`
	* Users can't use control center, we grey it out
	* Discouraged
* `.cooperative`
	* User can control
	* Your app can provide own UI
	* You observe changes to `isCenterStageEnabled`
	* Update your UI

## Portrait APIs
All devices with apple neural engine (2018+) (front-facing camera only)
All M1 Macs

Limitations:
* Computationally complex algorithm
	* Max res of 1920x1440
	* Max fps of 30

On/off state is sticky per app
User is always in control
Some apps need to opt in to use the feature

iOS voip apps: Automatically opted in
All other iOS apps must opt in `NSCameraPortraitEffectEnabled`
macOS: All apps automatically opted in

* Read-only instance properties
	* `captureDevice.activeformat.isPortraitEffectSupported`
	* `captureDevice.isPortraitEffectActive`

## Mic mode APIs
* Selected mode is sticky per app
* Users always in control
* Some apps need to opt in to use the feature

`AVCaptureDevice.MicrophoneMode`
* `.standard`
* `.wideSpectrum` (minimizes processing to capture all sounds, but still includes echo cancellation)
* `.voiceIsolation` (enhances speech, removes background noise)

`AVCaptureDevice.preferredMicrophoneMode`
`activeMicrophoneMode`.  Audio route may not support the preferred mode.

Available in:
All apps that use `AUVoiceIO`
2018 and later iOS/macOS

* You may prompt the user to disable or enable effects in control center
	* `AVCaptureDevice.showSystemUserInterface`.
	* Deeplinks into appropriate place

[[Capture high-quality photos using video formats]]


# Performance best practices
Use AVCapture classes to deliver a wide array of features.  
## AVCaptureVideoDataOutput
Avoid frame drops.  
`videoDataOutput.alwaysDiscardsLateVideoFrames = true`  Saves you from slow processing by always giving you the freshest frame and dropping frames you weren't ready to process.
Unless recording vide with `AVASsetWriter`.  So turn this off if you're doing that.

delegate callback: `captureOutput(didDrop:from:)`.

there is a dropped frame "reason".
* late: You are takign too long
* outOfBuffers: Holding on too many buffers
* discontintuity: system slowdown that's not your fault

Mitigation:
* Lower the device framerate dymamically
* `activeMinVideoframeDuration`
* Simplify processing

### System pressure
`AVCaptureDevice.SystemPressureState`
* `.Factors`
	* `.systemTemperature`
	* `.peakPower`: battery aging
	* `.depthModuleTemperature`: How hot the camera is getting
* `.Level`
	* `.nominal`
	* `.fair`: Slightly elevated pressure.  e.x. ambient temperature is high
	* `.serious`: Framerate throttling is advised
	* `.critical` => capture quality and performance are significantly impacted
	* `.shutdown` => system pressure is beyond critical.  Here, we stop automatically.

You can react to this in various ways
* Lower framerate
* Lessen workload on CPU/GPU
* Engage lower quality work modes
* AVCapture never degardes quality on its own, we don't know what you want
# IOSurface compression
CMSampleBuffer can wrap all kinds of media data as well as timing and metadata.

CVPixelBuffer: Speicifcally wraps pixel data and metadata attachments.

IOSurface: Memory to be wired to the kernel and provides interface for sharing large buffers across processes.

Lossless, in-memory video compression format
* Lowers overall memory bandwidth
* Understood by all hardware blocks
* Available on
	* iPHone 12 variants
	* Fall 2020 iPad Airs
	* Spring 2021 M1 iPad Pros

* Metal
* Vision (ANE)
* AVAssetReader/AVAssetWriter
* VideoToolbox
* Core Image
* CALayer

If AVCaptureSession doesn't need to deliver any buffers, your session is already taking advantage.

But if you want compressed surface dleivered to *your* output,

* Physical memory layout is opaque and may change
* Don't write to disk
* Don't assuem the same layout on all platforms
* Don't read/write using CPU

Uncompressed => compressed
* 420v => `&8v0` => `kCVPixelFormatType_Lossless_420YpCbCr8BiPlanarVideoRange`
* 420f => `&8f0` => `...FullRange`
* x420 => `&xv0` => `...BiPlanarVideoRange`
* BGRA => `&BGA` => `32BGRA`

## AVMultiCamPiP

# Wrap up
* Minimum focus distance
* 10-bit HDR video
* Video effects in control center
* Performance best practices
* IOSurface compression

* https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture



