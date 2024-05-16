Bring the latest advancements in Speech Synthesis to your apps. Learn how you can integrate your custom speech synthesizer and voices into iOS and macOS. We'll show you how SSML is used to generate expressive speech synthesis, and explore how Personal Voice can enable your augmentative and assistive communication app to speak on a person's behalf in an authentic way.


Voices can be personal.

# Explore Speech Synthesis Markup Language

W3C standard for speech
Declarative XML document
Supported on all Apple platforms
### 2:10 - SSML phrase
```html
<speak>
    Hello
    <break time="1s"/>
    <prosody rate="200%">nice to meet you!</prosody>
</speak>
```



### 2:29 - SSML utterance
```swift
let ssml = """
    <speak>
        Hello
        <break time="1s" />
        <prosody rate="200%">nice to meet you!</prosody>
    </speak>
"""

guard let ssmlUtterance = AVSpeechUtterance(ssmlRepresentation: ssml) else {
    return
}

self.synthesizer.speak(ssmlUtterance)
```


# Implement a synthesis provider
Synthesizer with great new voices.  iOS, macOS, iPadOS.  Speech synthesis providers allow you to implement your own speech synthesizer/voice.

Extension responsible for rendering audio to the SSML input, and optionally returning markers of when words occur.  System manages playback.


4-character identifier for app, 4-character identifier for vendor.


### 4:33 - Create a host app
```swift
struct ContentView: View {
    
    var body: some View {
        List {
            Section("My Awesome Voices") {
                ForEach(availableVoices) { voice in
                    HStack {
                        Text(voice.name)
                        Spacer()
                        Button("Buy") {
                            // Buy this voice...
                        }
                    }
                }
            }
        }
    }

    var availableVoices: [WWDCVoice] {
        return [
            WWDCVoice(name: "Screen Reader Voice", id: "com.example.screen-reader-voice"),
            WWDCVoice(name: "Reading Voice", id: "com.example.reading-voice")
        ]
    }   
}
```


### 5:04 - Keep track of purchased voices
```swift
struct ContentView: View {
    
    @State var purchasedVoices: [WWDCVoice] = []
    
    var body: some View {
        NavigationStack {
            List {
                MyAwesomeVoicesSection
                Section("Purchased Voices") {
                    ForEach(purchasedVoices) { voice in
                        NavigationLink {
                            // Destination View
                        } label: {
                            Text(voice.name)
                        }
                    }
                }
            }
        }
    }
}
```

### 5:13 - Inform the system when available voices change
```swift
struct ContentView: View {
    
    @State var purchasedVoices: [WWDCVoice] = []
    
    var body: some View {
        List {
            MyAwesomeVoicesSection
            PurchasedVoicesSection
        }
    }
    
    func purchase(voice: WWDCVoice) {
        // Append voice to list of purchased voices
        purchasedVoices.append(voice)
        
        // Inform system of change in voices
        AVSpeechSynthesisProviderVoice.updateSpeechVoices()
    }
}
```

`updateSpeechVoices` signals voices have changed.  System voice list will be rebuilt.

We can make this call after completing an IAP.

Keep tabs on which voices are available.

### 5:39 - Update UI with purchased voices
```swift
struct ContentView: View {
    
    @State var purchasedVoices: [WWDCVoice] = []
    
    var body: some View {
        List {
            Section("My Awesome Voices") {
                ForEach(availableVoices.filter { !purchasedVoices.contains($0) }) { voice in
                    HStack {
                        Text(voice.name)
                        Spacer()
                        Button("Buy") {
                            purchase(voice: voice)
                        }
                    }
                }
            }
            PurchasedVoicesSection
        }
    }
}
```

### 5:46 - Save available voices into UserDefaults
```swift
struct ContentView: View {
    
    let groupDefaults = UserDefaults(suiteName: "group.com.example.SpeechSynthesizerApp")!
    
    @State var purchasedVoices: [WWDCVoice] = []
    
    var body: some View {
        List {
            MyAwesomeVoicesSection
            PurchasedVoicesSection
        }
    }
    
    func purchase(voice: WWDCVoice) {
        // Append voice to list of purchased voices
        purchasedVoices.append(voice)
        
        // Write purchasedVoices to defaults
        updatePurchasedVoices()
        
        // Inform system of change in voices
        AVSpeechSynthesisProviderVoice.updateSpeechVoices()
    }
}
```

an app group allows us to share this voice group between app/extension.

Suite name - ensures host app and extension read from the same domain.

`AVSpeechSynthesizer` has new API to listen for change in system voices.  User can delete/download voices.  Subscribe to `availableVoicesDidChangeNotification`.

### 6:25 - Monitor for system voice changes
```swift
struct ContentView: View {

    @State var systemVoices: [AVSpeechSynthesisVoice] = AVSpeechSynthesisVoice.speechVoices()
    
    var body: some View {
        List {
            MyAwesomeVoicesSection
            PurchasedVoicesSection
            Section("System Voices") {
                ForEach(systemVoices.filter { $0.language == "en-US" }) { voice in
                    Text(voice.name)
                }
            }
        }
        .onReceive(NotificationCenter.default
            .publisher(for: AVSpeechSynthesizer.availableVoicesDidChangeNotification)) { _ in
                systemVoices = AVSpeechSynthesisVoice.speechVoices()
        }
    }
}
```

Inform the system of what voices we provide.  override `speechVoices` getter to provide a list of voices.  

For each item, we'll construct US english, AVSpeechSynthesisProvider Voice.



### 6:53 - Override speechVoices getter
```swift
// Implement a synthesis provider

public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override var speechVoices: [AVSpeechSynthesisProviderVoice] {
        get { }
    }
}
```

### 7:02 - Use UserDefaults to provide set of available voices
```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override var speechVoices: [AVSpeechSynthesisProviderVoice] {
        get {
            let voices: [String : String] = groupDefaults.value(forKey: "voices") as? [String : String] ?? [:]
            return voices.map { key, value in
                return AVSpeechSynthesisProviderVoice(name: value,
                                                identifier: key,
                                          primaryLanguages: ["en-US"],
                                        supportedLanguages: ["en-US"] )
            }
        }
    }
}
```


### 7:22 - Use your synthesis engine on each synthesis request
```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override func synthesizeSpeechRequest(speechRequest: AVSpeechSynthesisProviderRequest) {
        currentBuffer = getAudioBuffer(for: speechRequest.voice, with: speechRequest.ssmlRepresentation)
        framePosition = 0
    }
}
```



### 8:14 - Handle request cancellation
```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override func synthesizeSpeechRequest(speechRequest: AVSpeechSynthesisProviderRequest) {
        currentBuffer = getAudioBuffer(for: speechRequest.voice, with: speechRequest.ssmlRepresentation)
        framePosition = 0
    }

    public override func cancelSpeechRequest() {
        currentBuffer = nil
    }
}
```

audiounit fills the number of frames.

### 8:28 - Override internalRenderBlock
```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override var internalRenderBlock: AUInternalRenderBlock {
       return { [weak self]
           actionFlags, timestamp, frameCount, outputBusNumber, outputAudioBufferList, _, _ in
           guard let self else { return kAudio_ParamError }

           return noErr
       }
    }
}
```

### 8:42 - Implement the render block
```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override var internalRenderBlock: AUInternalRenderBlock {
       return { [weak self]
           actionFlags, timestamp, frameCount, outputBusNumber, outputAudioBufferList, _, _ in
           guard let self else { return kAudio_ParamError }

           // This is the audio buffer we are going to fill up
           var unsafeBuffer = UnsafeMutableAudioBufferListPointer(outputAudioBufferList)[0]
           let frames = unsafeBuffer.mData!.assumingMemoryBound(to: Float32.self)
                
           var sourceBuffer = UnsafeMutableAudioBufferListPointer(self.currentBuffer!.mutableAudioBufferList)[0]
           let sourceFrames = sourceBuffer.mData!.assumingMemoryBound(to: Float32.self)

           for frame in 0..<frameCount {
               if frames.count > frame && sourceFrames.count > self.framePosition {
                   frames[Int(frame)] = sourceFrames[Int(self.framePosition)]
                   self.framePosition += 1
                   if self.framePosition >= self.currentBuffer!.frameLength {
                       break
                   }
               }
           }
                
           return noErr
       }
    }
}
```

we set it to complete when we're done.  Signals to the system that rendering has completed and no more buffers to be rendered.
# Use Personal Voice

Record/recreate voice on iOS/macOS.  Personal voice generated on device, not on a server.  Voice will appear amongst rest of system voices.

Live speech - iOS, iPadOS, macOS, watchOS.

Type to speak on the fly.

Request access with these voices, using new auth API.  Use of personal voice is sensitive and should be primarily used for augmentative or alternative communication apps.


### 11:10 - Request authorization for Personal Voice
```swift
struct ContentView: View {

    @State private var personalVoices: [AVSpeechSynthesisVoice] = []

    func fetchPersonalVoices() async {
        AVSpeechSynthesizer.requestPersonalVoiceAuthorization() { status in
            if status == .authorized {
                personalVoices = AVSpeechSynthesisVoice.speechVoices().filter { $0.voiceTraits.contains(.isPersonalVoice) }
            }
        }
    }
}
```

Now that i have access to personal voice, I can use it to speak with.


### 11:34 - Use Personal Voice
```swift
func speakUtterance(string: String) {
    let utterance = AVSpeechUtterance(string: string)
    if let voice = personalVoices.first {
        utterance.voice = voice
        syntheizer.speak(utterance)
    }
}
```

# Next steps
Utilize SSML to create a rich speech experience

Implement unique voices to expand users' voice options
Bring a personal touch to your apps with personal voice.


# Resources
* https://developer.apple.com/documentation/audiounit
* https://developer.apple.com/documentation/avfaudio/audio_engine/audio_units/creating_an_audio_unit_extension
* https://developer.apple.com/documentation/avfoundation/speech_synthesis
* https://developer.apple.com/documentation/avfoundation/speech_synthesis
