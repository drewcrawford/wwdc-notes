Discover how you can bring AVFoundation, AVFAudio, and AudioToolbox to your apps on tvOS and create camera and microphone experiences for the living room. Find out how to support tvOS in your existing iOS camera experience with the Device Discovery API, build apps that use iPhone as a webcam or FaceTime source, and explore special considerations when developing for tvOS. We'll also show you how to enable audio recording for tvOS, and how to use echo cancellation to create great voice-driven experiences.

We'll discuss bringing camera/microphone support to tvOS.

Introduces continuity camera and mic to apple tv.  Stream to use as inputs on tvOS.  New genres of apps/experiences for big screen.

Most apps fall into two categories.
* playback
* games

enables you to build new apps for tvos like content creation apps, or social apps like video conferencing, streaming, etc.  Integrate camera/mic to existing streaming apps/games to create new features that weren't possible earlier.

[[Bringing continuity camera to your macOS app]]

# API overview
AVFoundation, AVFAudio / AudioToolbox.  Let's review how an app can use capture devices, specifically with AVFoundation's capture classes.

Apps use capture devices and inputs which represent camera and microphones.  Plug into AVCaptureSession.

Outputs - render data from inputs in various ways.  use these to record videos, snap photos, access buffers, metadata, etc.

AVCaptureVideoPreviewLayer - special output that subclasses CALayer.  Data flows through AVCaptureConnection.

Learn more in docs.

tvOS now supports same camera/mic apis from iOS.  Most of your code will just work on tvOS. 

since apple tv does nto have built in inputs, need to use device discovery.  Handle edge cases such as when devices appear/disappear.

Since the same camera/mic apis from iOS are available on tvos, let's walk through how to bring things over.

Next is capture session class which wraps several of the classes.  Property for avcapturesession, used to control which input is selected and where data is output.

ensure you have permission.  AVCaptureDevice.authorizationSTatus(for: .video).
AVCaptureDevice.requestAccess.

May be used to building apps for devices with built-in cameras.  apple tv is a communal device.  Best experience, everyone can use a device including guests.  Users can use recording features of your app at home, friend's house, shared space, etc.  cameras may appear/disappear at any time.

AVContinuityDevicePickerViewController.
* select an eligible contintuity device
* lists all signed in apple tv users
* mechanism for guest pairing

If one device is available, just start using it.  If none availalbe, use device picker.  

tvOS/iOS work together to ping eligible devices, prompt for confirmation.  User can accept notification on any notified device.

 AVContinuityDevicePickerViewController.  Delegate has lifecycle events, callbacks for when selected or available.  Captured devices are also published to discovery session, kvo etc.

must handle transitions from available/unavailable.

AVCaptureDevice.systemPreferredCamera.  Get the most suitable camera.  Updates automatically based on camera availability.  Sicne only one camera can be connected a time, nil is 0 cameras available, available is the only available camera.

can KVO the system preferred camera.

capture apis

| api | recommended usecase |
| ---- | ---- |
| AVCaptureDevice.systemPreferredCamera | get most suitable camera |
| AVCaptureDevice.DiscoverySession | enumerate captuer devices with availability change notifications |
| AVCaptureMetadataOutput | receive detected per-frame AVMetadataObjects (face, body, barcode detection) |
| AVCapturePhotoOutput | Capture high-res photo stills with controls like quality or speed prioritization, rotation or mirroring, and flash |
| AVCaptureMovieFileOutput | record movies with video/audio with controls like rotation, mirroring, resolution, and frame rate |
| Video fx / controls | check the status of video effects, adjust zoom factor and center stage behavior |
|  |  |
[[Capture high-quality photos using video formats]]
[[What's new in camera capture]]

# Continuity Camera

# Adapting apps for tvOS
user interaction
no direct touch events
focus-based
remote swipes, directional arrows, button presses

supports multiple users and guests
different file storage policies

## tvOS file storage
main consumers of disk space
* OS
* frameworks/app binaries
* cache for temporary data

mostly built for content consumption which requires a large cache.  Keeping with this disk space model ensures the best experience.

WRite to documentDirectory path is NOT recommended on tvOS.

No large persistant storage on tvOS
fails with a runtime error
Only use the `.cachesDirectory`.  Available while app is running.  POssible it will be purged between app launches.  Offload your data elsewhere ASAP.

[[Support multiple users in tvOS apps]]

Check out developer.apple.com.  
# Continuity microphone
use microphone on tvos for the first time ever.  Dive into what you need to do.

new APIs
AVFAudio - audio session

handle notifications like interruptions, etc.  full suite of recording apis brought over from iOS to tvOS.  These include recording apis in avfaudio as well as audiotoolbox.

apps can use a few different microphone devices on appletv.  continuity microphone, bluetooth devices like airpods/etc.  recognize type of input device via avaudiosession.

after going through device discovery, get access to continuity device.  with audioSessionPorts property.  Query from this property.

portType = .continuityMicrophone.  

you can continue using existing audio session api to query inputs on a system.  Existing portType carried over from iOS.

Microphone availability.  No built-in mic.  Your app is never guaranteed to always have access to a mic device.  inputAvailable now has KVO support to monitor when a mic device is available or not.

AVAudioApplication.shared.recordPermission
AVAudioApplication.requestRecordPermission().

Category/mode support.  Audio category and mode support for input routes.  Now available on tvOS.

Available in AVAudioSessionTypes.h


| API | recommended usecase |
| ---- | ---- |
| AVAudioRecorder | simplest way to record an audio file |
| AVCapture | Basic recording API accessible from AVCApture camera api |
| AVAudioEngine | Supports recording/playback, simplifies real-time audio and voice processing.  ex karaoke app. |
| AudioQueue | Low-level nonreal-time recording |
| AU remoteIO | Low-level interface for real-time audio |
| AU voiceio | low-level interface for RT audio and voice processing |
 for more info, see docs.
 recommended to opt into voice processing apis.
compared to standard echo cancellation problem, where recording and playback happens on same device.

recording takes place elsewhere from playback.  any arbitrary set of tv speakers, home theater setups, etc.  5.1, 7.1, run their own audio processing, etc.

user could be several feet away from mic device.  This mic device could be much closer to loud playback devices.  Challenging echo control problem to cancel out all playback while capturing local audio in hq.

voice processing apis
avAudioEngine.inputNode.setVoiceProcessingEnabled(true)

also via AU voiceio
`kAudioUnitSubType_voiceProcessingIO` subtype

see [[What's new in voice processing APIs]]
 
# Resources

https://developer.apple.com/documentation/avkit/supporting_continuity_camera_in_your_tvos_app