# When to leverage `AVSpeechSynthesizer`
*If you're trying to provide speech to everyone who uses your app*
* Maps apps uses speech to provide spoken directions

*If you're trying to provide speech only to people that use assistive technologies*
* VoiceOver is built in, no need to create screen reader
* Use UIAccessibility APIs to make your app accessible
* Use announcement notifications for extra speech

[[What's new in accessibility - 17]]
[[Deliver an exceptional accessibility experience - 18]]

```swift
UIAccessibility.post(notification: .announcement, argument: "Hello World")
```

## speech centric experiences
* Augmentative alternative commuication -> AAC
* Tool for people who are nonverbal

Blind/low-vision
* all UI still appropriately labeled and accessible
* `AVSpeechSynthesizer` helps describe real world environment
* More flexibility about when and how speech is spoken

# API refresher

* Synthesizer must be retained for the duration of speech
* Uses Spoken Content Settings by default
* Siri voices unavailable
	* will automatically fall back to a good voice if Siri is selected

```swift
self.synthesizer = AVSpeechSynthesizer()
let utterance = AVSpeechUtterance(string: "Hello World")
self.synthesizer.speak(utterance)
```

## Respecting assistive technology speech settings
* Applies speech settings from currently running assistive technology
* Takes precedence over properties set on the utterance
* Uses SCS when no assistive technology is running
* Not appropriate for all use cases, like AAC

```swift
self.synthesizer = AVSpeechSynthesizer()
let utterance = AVSpeechUtterance(string: "Hello World")
utterance.prefersAssistiveTechnologySettings = true
self.synthesizer.speak(utterance)
```

## cutomizaing speech
```swift
let utterance = AVSpeechUtterance(string: "Hello World")

// Choose a voice using a language code
utterance.voice = AVSpeechSynthesisVoice(language: "en-US")
        
// Choose a voice using an identifier
utterance.voice = AVSpeechSynthesisVoice(identifier: AVSpeechSynthesisVoiceIdentifierAlex)
        
// Get a list of installed voices
let voices = AVSpeechSynthesisVoice.speechVoices()
```

* voice list updates as more voices are downloaded in accesibility settings

```swift
let utterance = AVSpeechUtterance(string: "Hello World")

// Choose a rate between 0 and 1, 0.5 is the default rate
utterance.rate = 0.75
  
// Choose a pitch multiplier between 0.5 and 2, 1 is the default multiplier
utterance.pitchMultiplier = 1.5

// Choose a volume between 0 and 1, 1 is the default value
utterance.volume = 0.5
```

[[AVSpeechSynthesizer: Making iOS Talk - 18]]

# Mixing speech into an outside call

* Routes speech to outgoing call
* If no call is active, standard audio route is used
* great for AAC

```swift
self.synthesizer = AVSpeechSynthesizer()
self.synthesizer.mixToTelephonyUplink = true
```

opt out of application audio session
* delegates speech audio management to the system
	* audio interruptions handled
	* audio session set to active/inactive automatically
* speech will mix and duck with other audio

```swift
self.synthesizer = AVSpeechSynthesizer()
self.synthesizer.usesApplicationAudioSession = false
```

This way, we take care of certain things like interruptions, active/inactive

# wrap up
* speech should complement accessibility apis
* respect assistive technology settings
* consider including speech output in calls
* consider opting speech out of application audio sessions

