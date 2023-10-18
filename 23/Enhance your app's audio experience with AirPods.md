#airpods

Discover how you can create transformative audio experiences in your app using AirPods. Learn how to incorporate AirPods Automatic Switching, use AVAudioApplication to support Mute Control, and take advantage of Spatial Audio to create immersive soundscapes in your app or game.

deliver the best personal audio experience, etc.  Every day, many millions of people use airpods, etc.

how these features transform one of the most common usecases with airpods.  

Now you can press the airpod stem to mute yourself on the call.  You will be notified by a friendly microphone status banner, and an audio chime as well.

# AirPods automatic switching for macOS
supports airpods automatic switching between devices based on user's intended activity.
Uses indicators like 'now playing' registration and input audio activity

automatic participation for apps without sandbox restrictions or those using app sandbox

best practices.
now playing registration
make the right routing decisions.  If your app is a media or streaming app, register for now playing to prioritize audio.
If you are a conference or gaming app, we don't recommend registering for now playing.
we also recommend using audio services API to play app specific notification sounds.  Help differentiate notifications from media content and avoid unexpected behaviors.

conferencing paps -> record from the microphone only for the duration of a live meeting or voice session.

media apps -> use the selected default route to play audio
avoid playing silence after the user pauses audio
recommended to keep the duration of silence around 2s if required

# Press to Mute and Unmute

we are adding convenience while on call with press to mute/unmute.
New gesture on AirPods to control application's mic
callkit based apps will automatically get support
new API for communication apps not using CallKit

this feature adds significant convenience.  How quickly you can add on iOS 17.
* introducing AVAudioApplication.  New sibling in AVAudioSession family.
	* configure application-wide audio behaviors

### Press to Mute and Unmute API - 8:25
```swift
// Adopting AVAudioApplication into your App
import AVFAudio

// Get the started instance 
let instance = AVAudioApplication.shared

// Register for mute gesture notifications on Notification Center 
AVAudioApplication.inputMuteStateChangeNotification

// Key for mute state
AVAudioApplication.muteStateKey

// Updating AVAudioApplication’s mute state
instance.setInputMuted(...)

// Reading AVAudioApplication’s mute state
instance.isInputMuted
```

as you'd expect, we provide `setInputMuted`, etc.  That simple to incorporate press to mute/unmute on iOS 17.

Moving over to the mac.  Important to note that in macOS 14, press to mute and unmute works a little bit differently.

* apps are responsible for muting on macOS.
* additional adoption component in API
* please refer to [[What's new in voice processing APIs]] for new APIs for identifying muted talkers?

your app is responsible to mute on macOS, unlike iOS>


### Configure the Input Mute State Change handler (macOS only) - 10:52
```swift
// Configure the Input Mute State Change handler (macOS only)
instance.setInputMuteStateChangeHandler { isMuted in
	//...
	return didSucceed
}

// Optional: let CoreAudio mute your input for you (macOS only)
// Define the Core Audio property
var inputeMutePropertyAddress = AudioObjectPropertyAddress(
	mSelector: kAudioHardwarePropertyProcessInputMute,
	mScope: kAudioObjectPropertyScopeInput,
	mElement:kAudioObjectPropertyElementMain)

// Enable this property when you want to mute your input
UInt32 isMuted = 1; // 1 = muted, 0 = unmuted
AudioObjectSetPropertyData(kAudioObjectSystemObject,
						   &inputeMutePropertyAddress,
						   0,
						   nil,
						   UInt32(MemoryLayout.size(ofValue: isMuted),
						   &isMuted)
```

opportunity to reject if the mute action is not appropriate, etc.  Reocmmended that this handler should not be used for UI updates since you wll continue to receive inputMuteStateChange notification when mute state changes.

In addition, we have provided a new property within core audio to help you quickly mute any input audio to your process.  This property when enabled will silence any input audio to your process while continuing IO as usual.
# Spatial audio with AirPods

over 80% of apple music subscribers listen in spatial audio.  iOS 16, we took this to the next level with personalized spatial audio.  Based on size and shape of head and ears.  This push towards increased personalization has been well-received.  we are glad to announce continued spatial audio support for all 3 platforms.

| macOS                       | tvOS                        | iOS                         |
| --------------------------- | --------------------------- | --------------------------- |
| AVPlayer                    | AVPlayer                    | AVPlayer                    |
| AVSampleBufferAudioRenderer | AVSampleBufferAudioRenderer | AVSampleBufferAudioRenderer |
| ?                           | audioqueue                  | audioqueue                  |
| ?                           | AuRemoteIO                  | AuRemoteIO                            |

some of these when registered for nwo playing with user control to disable
* spatial audio for aQ and AuremoteIO does not have an API interface
* automatically enabled for apps that register for now playing
* provides the user control the feature using control center

# Wrap up
* transform your app behavior for airpods automatic switching
* consistent automatic switching experience with airpods
* Press to mute and unmute and the associated api
* spatial audio across platforms

[[Immerse your app in spatial audio]]
[[Becoming a Now Playable App]]




