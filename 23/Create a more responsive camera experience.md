Discover how AVCapture and PhotoKit can help you create more responsive and delightful apps. Learn about the camera capture process and find out how deferred photo processing can help create the best quality photo. We'll show you how zero shutter lag uses time travel to capture the perfect action photo, dive into building a responsive capture pipeline, and share how you can adopt the Video Effects API to recognize pre-defined gestures that trigger real-time video effects.

#avfoundation #photos 

# Deferred photo processing
responsiveness  prior to iOS 17
AVCapturePhotoSEttings.photoQualityPrioritization

* .speed
* `.balanced`
* `.quality`

we can enable the new featuers and go through concepts 1 by 1.

quality may use deep fusion on iPhone 11 pro+.  This must complete before next capture request will start.

* synchronous processing
* impacts shot-to-shot time
* may miss the next 'moment'

AVCapturePhotoOutputDelegate
`willBeginCaptureFor`.  Use our procesisng tehcniques to fuse them intoa  deep fusion iamge.

sends photo back via `didFinishProcessingPhoto` callback.  Must complete before next capture happens.

call capturePhoto before callback fires, but it won't start until previous processing is complete.

with dferred processing, you request a photo and when suitable, it will deliver the proxy photo.  

when configuring an aVCaptureSession, you

1.  add device input
2. AVCapturePhotoOutput
3. select particular format or session preset

final photo processing happens either

1.  on demand, when you get data back from library
2. in background, such as when system is idle

wgheb we use addResource we use `photoProxy` type.  this tells us to do photo processing.

`requestImageForAsset`.  The first callback will hold a lower-resolution image, while the final image callback will not.  I can make a check for those here.  `PHImageResultIsDegradedKey`

receive a secondary image from the `requestImageForAsset` method.  This lets us show an image in the meantime during photo processing.  We use `allowSecondaryDegradedImage` for this.

New image will hold existing key `PHImageResultIsDegradedKey`.

I guess we did this previously, but now we have more intermediate images while the final image processes.

adoption.
* must have write access tot he photo library to store the proxy photo.  And read permission if you want to see the final photo.  Only request the smallest amount of access needed.
* Get the proxy into the library ASAP.  When you're backgrounded, you have a limited amount of time to run.  If memory pressure is too great, you might be forcequit.  Getting this into the library ensures minimal chance of data loss.
* Final image or metadata changes must be handled as adjustments using PhotoKit
* handle both deferred and non-deferred photos int he same session
* not available on all photos, such as flash
* No AVCapturePhotoSEttings opt-in property
* user experience.  Deferred photo processing provides our best image quality with rapid shot-to-shot times. 
* available on iPHone 11Pro+.
[[AvCapturePhotoOutput - beyond the basics]]

# Zero shutter lag

we keep a rolling ring buffer of frames seen in the past.  So we can do time travel and get the photo at the exact right time.

apps linked after iOS 17 are opted in.  Nothing to do!
if you're not getting it, set `.isZeroShutterLagEnabled` and `isZeroShutterLagSupported`.

not available for flash capture, manual-exposure photo, bracketp hotos, or constitutent photo delivery, are not available.

minimize effects of camera shake by capturing as soon as possible.
# Responsive capture
resonsive capture - 4fps vs 2fps.  

capture photo
process photo
encode photo

we can pipeline and overlap these phases, so that a new capture phase can start while a previous photo is in the processing phase.  Note that this increases peak memory.  This puts pressure on the system, so you may need to opt out.


note that because we are pipelining, you might get delegate callbacks for different photos that are overlapped in pielining.

`AVCapturePHotoOutput.isZeroShutterLagEnabled`.  Then `isResponsiveCaptureSupported`.  Then turn it on by setting `isResponsiveCaptureEnabled = true`.  

## fast-capture prioritization
Detects multiple, fast captures
prioritizes consistent shot-to-shot times by adapting photo quality.
Since this impacts quality it is off by default.  This is called "prioritize faster shooting" in the camera app.  We think the consistent shot-to-shot times are more important.

`isFastCapturePrioritizationSupported` and `isFastCapturePrioritizationEnabled` properties

AVCapturePhotooutput.CaptureReadiness

`.notRunning` `ready`, and 3 not ready states.

use `AVCapturePhotoOutputReadinessCoordinator`.  Use this class even if you don't use resonsive capture or fast capture prioritization APIs.

* ready - user interaction enabled
* notReadyWaitingForProcessing - consider using a progress bar here.

iPHones with a12 bionic chip and newer.  iPhone XS and newer?

use just the ones that are appropriate for your app.
# Video effects

previously control center offered options.  center stage, portrait, studio light, etc.

now we've moved this into its own menu.  

## reaction effects
* system-level camera feature
* avialble to your apps out of the box
*  no code changes needed
[[What's new in camera capture]]

can click on react effect in video effects menu in macOS
or call `AVCaptureDevice.performEffect`, ex your own UI.
they can be sent by making a gesture.

fireworks (two thumbs up)
heart
balloons with one victory sign
rain with two thumbs down
confetti with two victory signs
lasers using two metal signs

`reactionEffectsSupported`p roperty.

`canPerformREactionEffects` and `reactionEffectGesturesEnabled`.  Remember, because these rae user controlled, you can't turn on or off.

you can KVO.  To trigger on iOS, you need to do this programmatically.  

AVCaptureReactionType.systemImageName().  get the image for these.  Use with `UIImage(systemName:)`.

`AVCaptureDevice.reactionEFfectsInProgress`.  They may overlap, so this returns an array.

can use KVO to know when they begin/end.  VoIP apps should send this as meetadata to remote views.  ex, if video is turned off.

this may be challenging to video encoder, need more bandwidth.  via KVO you can increase bitrate while effects are rendering, or in videotoolbox, avoid low values for kVTCompressionPropertyKey_MaxAllowedFrameQP

[[Explore low-latency video encoding with VideoToolbox]]

framerate may change while effects are in progress.  I think we throttle these to 30fps.  This follows the model of portrait and studio light effects, where the framerate may be lower than you specify

`videoFrameRAteRangeForReactionEffectsInProgress`. informational-only, can't control it.

macOS, tvOS - always enabled
iOS - voip application category, or NSCameraReactionEFfectsEnabled.

available on A14 and newer, apple silicon macs.  Others are complicated.

# Wrap up
* make a mroe ersponsive photography app with new APIs
* make video calls expressive with updated video effects
* 
# Resources
* https://developer.apple.com/library/content/samplecode/AVCam
* https://developer.apple.com/documentation/avfoundation
* https://developer.apple.com/av-foundation/
