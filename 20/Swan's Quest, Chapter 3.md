```swift
// Example Pitch implementation

public enum Pitch: Double, PitchProtocol {
    case a4 = 440.0
    
    var frequency: Double {
        return self.rawValue
    }
}
```

```swift
// Music.swift

public protocol NoteProtocol {
    
    /// Play this Note through a ToneOutput
    var tone: Tone { get }
    
    /// The duration of this Note as a multiple of quarter notes,
    /// e.g., a half note is 2.0, an eighth note is 0.5
    var length: Float { get }
}
```

```swift
// Example Note implementation

public enum Note: NoteProtocol {
    case quarter(pitch: Pitch)
    
    var tone: Tone {
        switch self {
        case .quarter(let pitch):
            return Tone(pitch: pitch.frequency, volume: 0.3)
        }
    }
    
    var length: Float {
        switch self {
        case .quarter(_):
            return 1.0
        }
    }
}
```

```swift
// Play more than one tone redux

let toneOutput = ToneOutput()
let notes = [Note.quarter(pitch: .a4), .half(pitch: .c4), .quarter(pitch: .a4)]

var index = 0
Timer.scheduledTimer(withTimeInterval: 0.4, repeats: true) { timer in
    guard index < tones.count else {
        timer.invalidate()
        owner.endPerformance()
        return
    }
    
    toneOutput.play(tone: tones[toneIndex].tone)
    index += 1
}
```

```swift
//updating the Note protocol
// Music.swift

public protocol NoteProtocol {
    
    /// Play this Note through a ToneOutput
    var tone: Tone { get }
    
    /// The duration of this Note as a multiple of quarter notes,
    /// e.g., a half note is 2.0, an eighth note is 0.5
    var length: Float { get }

    /// Length of the smallest Note supported
    static var shortestSupportedNoteLength: Float { get }
}
```

```swift
// Play more than one tone redux

let toneOutput = ToneOutput()
let notes = [Note.quarter(pitch: .a4), .half(pitch: .c4), .quarter(pitch: .a4)]
var index = 0

let interval = TimeInterval(Note.shortestSupportedNoteLength * 0.5) // 120 BPM
Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { timer in
    guard index < tones.count else {
        timer.invalidate()
        owner.endPerformance()
        return
    }
    
    toneOutput.play(tone: tones[toneIndex].tone)
    index += 1
}
```

```swift
//adding subidivide to Note protocol
// Music.swift

public protocol NoteProtocol {
    associatedtype PitchType: PitchProtocol
 
    /// Play this Note through a ToneOutput
    var tone: Tone { get }
    
    /// The duration of this Note as a multiple of quarter notes,
    /// e.g., a half note is 2.0, an eighth note is 0.5
    var length: Float { get }

    /// Length of the smallest Note supported
    static var shortestSupportedNoteLength: Float { get }

    /// Subdivide into a series pitches, according to the shortest
    /// supported note
    func subdivide() -> [PitchType]
}
```

```swift
//putting it all together
// Play more than one tone redux
    
let toneOutput = ToneOutput()
let notes = [Note.quarter(pitch: .a4), .half(pitch: .a4), .quarter(pitch: .a4)]
var pitches = [Pitch]()
for note in notes {
    pitches.append(contentsOf: note.subdivide())
}
var index = 0

let interval = TimeInterval(Note.shortestSupportedNoteLength * 0.5)
Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { timer in
    guard index < pitches.count else {
        timer.invalidate()
        owner.endPerformance()
        return
    }
    toneOutput.play(tone: Tone(pitch: pitches[index].frequency, volume: 0.3))
    index += 1
}
```

