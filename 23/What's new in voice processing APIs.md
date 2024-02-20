Learn how to use the Apple voice processing APIs to achieve the best possible audio experience in your VoIP apps. We'll show you how to detect when someone is talking while muted, adjust ducking behavior of other audio, and more.

These clean up acoustic environments.

Best-in-class audio signal processing.
Performance is tuned per-product and device type.
Grant user full control over the mic modes - standard, voice isolation, wide spectrum, etc.

Two options
* AUVoiceIO
	* for apps that interact with I/O audio unit directly
* AVAudioEngine voice processing mode
	* higher-level API, easier to use, less code

# What's new
* voice processing available on tvOS [[Discover Continuity Camera for tvOS]]
* New APIs
	* Other audio ducking configuration
	* Muted talker detection
## Other audio ducking

Your app provides audio stream played to output device.  But there may be other audio streams playing.  You might be playing through some other API. Or other apps play audio at the same time as yours.    All these are "other audio" to voice processing.

In the past, we applied a fixed amount of ducking to other audio.  If you're happy, don't do anything.  However, we realized that some apps like to have more control over ducking behavior.

Two independent aspects of ducking.

mEnableAdvancedDucking
mDuckingLevel.

Default, minimum (loudest), medium, maximum (quietest).  Choosing a higher ducking level improves intelligibility of the voicechat.


### 5:50 - Other audio ducking
```c
struct AUVoiceIOOtherAudioDuckingConfiguration {
	Boolean mEnableAdvancedDucking;
	AUVoiceIOOtherAudioDuckingLevel  mDuckingLevel;
};

typedef CF_ENUM(UInt32, AUVoiceIOOtherAudioDuckingLevel) {
	kAUVoiceIOOtherAudioDuckingLevelDefault = 0,
	kAUVoiceIOOtherAudioDuckingLevelMin = 10,
	kAUVoiceIOOtherAudioDuckingLevelMid = 20,
	kAUVoiceIOOtherAudioDuckingLevelMax = 30
};
```

Create a ducking configuration.
### 6:48 - Other audio ducking
```cpp
const AUVoiceIOOtherAudioDuckingConfiguration duckingConfig = {
	.mEnableAdvancedDucking = true,
	.mDuckingLevel = AUVoiceIOOtherAudioDuckingLevel::kAUVoiceIOOtherAudioDuckingLevelMin
};
// AUVoiceIO creation code omitted
OSStatus err = AudioUnitSetProperty(auVoiceIO, kAUVoiceIOProperty_OtherAudioDuckingConfiguration, kAudioUnitScope_Global, 0, &duckingConfig, sizeof(duckingConfig));
```

### 6:50 - Other audio ducking
```cpp
const AUVoiceIOOtherAudioDuckingConfiguration duckingConfig = {
	.mEnableAdvancedDucking = true,
	.mDuckingLevel = AUVoiceIOOtherAudioDuckingLevel::kAUVoiceIOOtherAudioDuckingLevelMin
};
// AUVoiceIO creation code omitted
OSStatus err = AudioUnitSetProperty(auVoiceIO, kAUVoiceIOProperty_OtherAudioDuckingConfiguration, kAudioUnitScope_Global, 0, &duckingConfig, sizeof(duckingConfig));
```

AVAudioEngine.
### 7:20 - Other audio ducking
```swift
public struct AVAudioVoiceProcessingOtherAudioDuckingConfiguration {
	public var enableAdvancedDucking: ObjCBool 
	public var duckingLevel: AVAudioVoiceProcessingOtherAudioDuckingConfiguration.Level
}
extension AVAudioVoiceProcessingOtherAudioDuckingConfiguration {
	public enum Level : Int, @unchecked Sendable {
		case `default` = 0
		case min = 10
		case mid = 20
		case max = 30
	}
}
```

### 7:31 - Other audio ducking
```swift
let engine = AVAudioEngine()
let inputNode = engine.inputNode
do {
	try inputNode.setVoiceProcessingEnabled(true)
} catch {
	print("Could not enable voice processing \(error)")
}
let duckingConfig = AVAudioVoiceProcessingOtherAudioDuckingConfiguration(mEnableAdvancedDucking: false, mDuckingLevel: .max)
inputNode.voiceProcessingOtherAudioDuckingConfiguration = duckingConfig
```

Muted talker detection.  API to detect presence of a muted talker.  Introduced in iOS 15, now we're making it available in macOS 14 and tvOS 17.

Provide a listener block to AUVoiceIO etc. to receive the notification.

Implement your handling of this notification in your listener block
Implement mute via the mute API.
### 7:32 - Muted talker detection AUVoiceIO
```cpp
AUVoiceIOMutedSpeechActivityEventListener listener = 
    ^(AUVoiceIOMutedSpeechActivityEvent event) {		
        if (event == kAUVoiceIOSpeechActivityHasStarted) {
            // User has started talking while muted. Prompt the user to un-mute
        } else if (event == kAUVoiceIOSpeechActivityHasEnded) {
            // User has stopped talking while muted
        }
    };
OSStatus err = AudioUnitSetProperty(auVoiceIO, kAUVoiceIOProperty_MutedSpeechActivityEventListener, kAudioUnitScope_Global, 0, &listener,  sizeof(AUVoiceIOMutedSpeechActivityEventListener));
// When user mutes
UInt32 muteUplinkOutput = 1;
result = AudioUnitSetProperty(auVoiceIO, kAUVoiceIOProperty_MuteOutput, kAudioUnitScope_Global, 0, &muteUplinkOutput, sizeof(muteUplinkOutput));
```

### 11:08 - Muted talker detection AVAudioEngine
```swift
let listener = { (event : AVAudioVoiceProcessingSpeechActivityEvent) in
    if (event == AVAudioVoiceProcessingSpeechActivityEvent.started) {
        // User has started talking while muted. Prompt the user to un-mute
    } else if (event == AVAudioVoiceProcessingSpeechActivityEvent.ended) {
        // User has stopped talking while muted
    }
}
inputNode.setMutedSpeechActivityEventListener(listener)
// When user mutes
inputNode.isVoiceProcessingInputMuted = true
```

Alternative macOS APIs.  Only available on macOS.

two new HAL properties.

`kAudioDevicePropertyVoiceActivityDetectionEnable` on input device.
`kAudioDevicePropertyVoiceActivityDetectionState` to register listener.  This is invoked whenever there's a change.

When notified, query the detection state for its current value.
### 12:31 - Voice activity detection - implementation with HAL APIs
```cpp
// Enable Voice Activity Detection on the input device
const AudioObjectPropertyAddress kVoiceActivityDetectionEnable{
        kAudioDevicePropertyVoiceActivityDetectionEnable,
        kAudioDevicePropertyScopeInput,
        kAudioObjectPropertyElementMain };
OSStatus status = kAudioHardwareNoError;
UInt32 shouldEnable = 1;
status = AudioObjectSetPropertyData(deviceID, &kVoiceActivityDetectionEnable, 0, NULL, sizeof(UInt32), &shouldEnable);
// Register a listener on the Voice Activity Detection State property
const AudioObjectPropertyAddress kVoiceActivityDetectionState{
        kAudioDevicePropertyVoiceActivityDetectionState,
        kAudioDevicePropertyScopeInput,
        kAudioObjectPropertyElementMain };
status = AudioObjectAddPropertyListener(deviceID, &kVoiceActivityDetectionState, (AudioObjectPropertyListenerProc)listener_callback, NULL); // “listener_callback” is the name of your listener function
```



### 13:13 - Voice activity detection - listener_callback implementation
```cpp
OSStatus listener_callback(
    AudioObjectID                 inObjectID,
    UInt32                        inNumberAddresses,
    const AudioObjectPropertyAddress*   __nullable inAddresses,
    void* __nullable              inClientData)
{
  // Assuming this is the only property we are listening for, therefore no need to go through inAddresses
       UInt32 voiceDetected = 0;
     UInt32 propertySize = sizeof(UInt32);
     OSStatus status = AudioObjectGetPropertyData(inObjectID, &kVoiceActivityState, 0, NULL, &propertySize, &voiceDetected);
  
       if (kAudioHardwareNoError == status) {
 if (voiceDetected == 1) {
    // voice activity detected
	} else if (voiceDetected == 0) {
		    // voice activity not detected
	}
 }
 return status;
};
```

Important details about these HAL APIs
* detection of echo-canceled voice activity
* Detection regardless of process mute state
* To implement muted talker detection, needs additional logics to combine with mute state.
* AHL process mute API `kAudioHardwarePropertyProcessInputMute`.  This supresses light in the menu bar and gives users privacy confidence
# summary
* apple voice processing APIS
* other audio ducking configuration
* AUVoiceIO, AVAudioEngine
* Muted talker detection
* AUVoiceIO, AVAudioEngine
* macOS HAL
