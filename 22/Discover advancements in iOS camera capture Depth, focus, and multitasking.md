#camera 

# LiDAR scanner
Streaming depth data on top of the camera feed demo.

API firs titnroduced in ARKit iPadOS 13.4
[[Explore ARKit 4]]

New in IOS 15.4, your app can access with aVFoundation `AVCaptureDevice.DeviceType`
Delivers video and depth
High-quality high-accuracy depth info

Uses wide-angle camera with LiDAR scanner.
Wide-angle camera's FOV
All formats support depth

Quality from fusing sparse LiDAR with color image
Outputs dense depth map
Multi-cam with Telephoto or Ultra Wide devices

many formats.  Not writing them down, but theres's a table.

availability:
* iPhone 12 Pro
* iPhone 13 pro
* Ipad pro 5gen

combinatin of cameras.  These are virtual devices, that consists of physical devices.  There are 4 devices.

Differences in the type of depth these produce
* LiDAR depth camera
	* absolute depth
	* real-world scale
	* great for computer vision
* TrueDepth, Dual, Dual Wide, and Triple
	* Relative, disparity-based depth
	* less power
	* great for photo effects

* representation for depth data
* Delivered by a depth-capable AVCaptureDevice
* Stream from AVCaptureDepthDAtaOutput
* Attached to photos from aVCapturePhotoOutput
Depth data is filtered by default
Reduces noise and fills in missing values (holes)
Great for video and photography apps

For computer vision, prefer non-filtered
LiDAR depth camera excludes low confidence poitns

To disable filtering, set `isFilteringEnabled = false`.  When you receive a depth data object, it will not be filtered.

## Which framework to use
| avfoundation          | ARKit             |
| --------------------- | ----------------- |
| video and photography | augmented reality |
| Higher resolution     | AR algorithms     |
|                       |                   |

See [[Capturing depth in iPhone Photography - 17]]

# Face-driven AF/AE
These systems analyze scene.  Focus => adjusts lens.  Exposure => balances brightest/darkest to keep visible.

A common feature of DSLRs is to track faces in the scene to dynamically adjust and keep visible.

In iOS 15.4, we prioritize faces.  Enabled by default for all apps on iOS 15.4 oer later.

Notice that by keeping his face well-exposed, trees appear brighter, etc.  Exposure of the scene is adjusted.

In iOS 15.4, there are new proeprties on AVCaptureDevice to control whether this is eanbled.  `automaticallyAdjustsFaceDrivenAutoFocusEnabled` etc.  NOte that are two levels: whether the feature will be automatically enabled, and whether it is enabled.

* great for photography
* Used by Camera app
And for video conferencing
* used by FaceTime

Not suited for all cases
* Manual control

1.  Lock for configuration
2. Turn off automatic adjustments
3. Disable the features themselves
4. Unlock configuration

# advanced streaming
AVFoudnation capture allows developers to build immersive apps with the camera.
* AVCaptureSession manages data flow
* AVCaptureInput provides data
	* cameras and microphones, etc.

ex: applying filters to video

1.  Camera/mic inputs
2. Video/audio outputs
3. Data has effects applied
4. Sent to preview, and writer
5. audio to writer alone

New, apps can use multiple outputs at the same time.  For each video data output, you can customizae

* resolution
* stabilitzation
* orientation
* pixel format

This example is balancing preview vs recording.  For preview, we need screen resolution and fast processing.  For recording, best to capture high resolution with quality effects applied.  By adding a second output, the graph can be extended.  Now we can do one output at 1080p for preview, another one at 4k for recording.

Another reason to use separate vide outputs is to apply different stabilization modes.  Additional latency to the pipeline.  For preview, maybe we don't want any?  For recording, stabilization can be applied.  

## Configuring video resolution
full-size (by default in most cases)
1. disable output buffer dimensions
2. disable preview sized output buffers

Preview-sized (default for photo preset)
1.  Disable autoamtic output dimensions
2. enable preview sized buffers

Custom resolution:
1.  video settings dictionary
2. aspect ratio must match source device

more ways
* stabilization `(.preferredVideoSTabmilizationMode)`
* orientation (`.videoOrientation`
* pixel format

For more information, see the docs and tech note.

## Movie file with video and audio data
Check suported combinations with `.canAddOutput` and `hardwareCost`.  By receiving video data with movie fil eoutput , can inspect while recording and analyze scene.

audio data: sample audio while recording, listen ot what is being recorded.  

Implementing these advanced streaming ocnfigurations requires the use of new new API.
Do more with existing API.




# Multitasking camera access
On iPad, users, can multitask in many ways.  record notes in splitview, slideover notes, etc.
stage manager, etc.
STarting in iOS 16, AVCaptureSession will be able to use while multitasking.  We presented this before because of QoS.

Apps compete for system resources, e.g. games while using camera.  User watching a video months/years later may not remember they recorded it while multitasking.  

When the system detects video was recorded during multitasking, will alert user.  Shown only once by the system and will have an ok button to dismiss.

`.isMultitaskingCameraAccessSupported` and `enabled`.  If enabled, you will not be errored for multitasking.

You might want afullscreen experience.  e.g. ARKit does not support multitasking.  Ensure your app performs well alongside other apps.  Responding to system pressure notifications.  Lowesring frame rate.  Choosing less demanding video formats.  Lower-resolution, binned, or non-HDR formats.

See the article.  

Now you can display participants in a pip window.  `activeVideoCallSourceView` etc.  To learn more, see the article.

# Wrap up
* LiDAR scanner
* Face-driven AF/AE
* ADvanced streaming
* Multitasking camera access


* https://developer.apple.com/forums/tags/wwdc2022-110429
* https://developer.apple.com/forums/create/question?&tag1=281&tag2=137&tag3=531030
* https://developer.apple.com/documentation/Technotes/tn3121-selecting-a-pixel-format-for-an-avcapturevideodataoutput
* https://developer.apple.com/documentation/avkit/adopting_picture_in_picture_for_video_calls
* https://developer.apple.com/documentation/avkit/accessing_the_camera_while_multitasking
* https://developer.apple.com/documentation/avfoundation/capture_setup
