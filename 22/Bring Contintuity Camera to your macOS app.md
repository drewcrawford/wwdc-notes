# What is continuity camera?
iPhone appears on your mac as external camera/mic.
* must be running macOS 13, iOS 16.  Sign into apple id using 2fa.
* For fired connection, must use usb.
* For wireless connection, proximity.  Both bluetooth and wifi turned on.

## Demo
One-time dialog after upgrading.  When you open app and there's an eligible iPhone.

After the dialog is shown, camera is available in all apps.  

Uses rear camera system on iPhone, so you get the same great video quality.  Works with all four orientations of the phone.  Portrait orientation gives you a more zoomed in FOV compared to landscape.

Several new video effects.
portrait, center stage.
[[What's new in camera capture]]

with continuity camera, portrait is now available on intel as well.  
studio light => new on macOS 13.  When using iPhone 12 or newer.  Perfect for tough lighting situations, such as in front of a window.  Even though I'm showing each ffect separately, all of them work well together.

Any combination of the effects can be applied at the same time.

Can now use Desk View.  Find in control center.  Collaborate on a project, etc.  Extended vertical field of view of UW angle camera.  Applies perspective distortion, and rotates the frames to create the desk view.  Share window function available in most video conferencing apps to share this feed in parallle with the main video feed.

Can also be used alone without streaming from the main camera.  When you stream from obht, we reocmmend center stage for better framing on the main camera.

jAlso a desk view camera API to provide customized integration for your app.  

All notifications on your phone will be silenced, and important call notifications will be forwarded on your mac.


# Building a magical experience
Wth some new APIs you can make continuity even more magical in your app.

Prior to macOS 13, when a device is unplugged, a manual selection step is usualy required in applications.

We'd like you to switch cameras automatically.  Two new APIs iun AVFoudnation framework to build this in
`userPreferredCamera` and `systemPreferredCamera`

* user
	* read-write property
	* set this whenever a user picks a camera in app
	* lets macOS learn user preference
	* tracks connect/disconnect
	* KVO
	* always tries to return a device that's ready to use.
	* Only returns nil when there is no camera available
* system
	* read-only
	* Incorporates userPreferredCamera and a few other factors
	* different property whena  continuity camera shows up
	* tracks device suspensions internally so it prioritizes unsuspended devices
	* helpful for building automatic switching behavior, e.g. closing macbook lid

Always KVO these properties and update your capture session accordingly.  

These APIs are adopted by first-party application.  Provide customers a universal and consistent method of camera selection.

In FT, we enable "automatic camera selection".  We recommend adding new UI for enabling/disabling.

## Demo
system camera changes to ocntinuity camera when phone is ready to stream.  maybe build your app in a similar way.

* automatic camera selection is on
	* start kvo system camera
	* update sesion's input device
	* set userPreferredCAmera when user picks a camera
		* will be reflected in systemPreferredCamera
* when off
	* stop KVO
	* update session's input with user selection
	* set user camera when user picks a camera

For best practices, see the sample code.


# New APIs on macOS
New opportunities to harness iPhone-specific camera features in your mac app.

## iPhone photo captures to macOS.
High-resolution photo capture: 12mp.  Previously, we only supported video resolution.  Now we support 12mp.

`isHighResolutionCaptureEnabled`
and `isHighREsolutionPhotoEnabled` for each capture

## Photo quality prioritization
`.maxPHotoQualityPrioritization`
`.photoQualityPrioritization` for each capture.

[[Capture high-quality photos using video formats]]

## Flash capture
`photoSettings.flashMode`.

## AVCaptureMetadataOutput
now stream face metadata and human body metadata.

sample code not included

## Continuity camera capabilities
* photo output
* metadata output
* video data output
* movie file outputs
* preview layer

contintuity camera formats.  Table not included.  Three 16x9 formats, 1 4x3.  Choose between formats supporting up to 30fps or 60fps based on need.

## Desk view
Separate AVAcpatureDevice.  Two wasy to find it.  First, look up `AVCaptureDeviceType` `.deskViewCamera`.

Alternatively, if you know the main camera, you can use `.companionDeskViewCamera` on it.  

* video data output
* movie file output
* video preview layer

desk view supports 1920x1440 1-30fps in 420v.  

# Wrap up
* continuity camera
* Building a magical experience
* New APIs

* https://developer.apple.com/documentation/avfoundation/capture_setup/supporting_continuity_camera_in_your_macos_app
* https://developer.apple.com/documentation/avfoundation/capture_setup
