Bring the latest advancements in Speech Synthesis to your apps. Learn how you can integrate your custom speech synthesizer and voices into iOS and macOS. We'll show you how SSML is used to generate expressive speech synthesis, and explore how Personal Voice can enable your augmentative and assistive communication app to speak on a person's behalf in an authentic way.

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

# Resources
* https://developer.apple.com/documentation/audiounit
* https://developer.apple.com/documentation/avfaudio/audio_engine/audio_units/creating_an_audio_unit_extension
* https://developer.apple.com/documentation/avfoundation/speech_synthesis
* https://developer.apple.com/documentation/avfoundation/speech_synthesis
