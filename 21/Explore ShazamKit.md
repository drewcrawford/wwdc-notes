#shazamkit
# Introducing ShazamKit
Regoznies audio, exact matches to audio in catalog
Custom audio catalogs you create!

SoundAnalysis - ues this one to categorize speech, humming, laughter, applause, etc.

[[Discover built-in sound classification in SoundAnalysis]]

* Shazam catalog recognition
* Custom catalog recognition – on-device matching against arbitrary audio
* library management

[[Create custom audio experiences with ShazamKit]]

Make your experience more seamless by using contents audio as a remote control, trigger synced activities.   

Interactive 
# Use case examples
Shazam matches against signature a lossy representation of a song.

* Smaller than the audio
* Audio cannot be reconstructed, protecting privacy
*
Shazam catalog - collection of reference signatures.  Hosted/maintained by apple.

Reference signature, query signature.

```swift
// Matching signatures using SHSession
let session = SHSession()
session.delegate = self

let signatureGenerator = SHSignatureGenerator()
try signatureGenerator.append(buffer, at: nil)

let signature = signatureGenerator.signature()
session.match(signature)
```

Use streaming API if needed.

```swift
// Receiving matches via the session delegate
extension SongResultViewController: SHSessionDelegate {
    
    public func session(_ session: SHSession, didFind match: SHMatch) {
        
        guard let matchedMediaItem = match.mediaItems.first else {
            return
        }
        
        DispatchQueue.main.async {
            self.songView.titleLabel.text = matchedMediaItem.title
            self.songView.artistLabel.text = matchedMediaItem.artist
        }
       
    }
}
```

```swift
// Adding to a customer’s library
guard let matchedMediaItem = match.mediaItems.first else {
    return
}

SHMediaLibrary.default.add([matchedMediaItem]) { error in
    
    if error != nil {
        
        // handle the error
    }
        
}
```

No special permission to access the shazam library, but avoid storing content in it without making customers aware.  All things stored will be attributed to the app.

` matchOffset` => where the song was made

MusicKit provides strongly-typed objects for songs.

[[Meet MusicKit for Swift]]


# Getting started
Consider these best practices
* Prefer `matchStreamingBuffer` for real-time audio
* Consider microphone usage (reducing it, such as by stopping when you have a match)
* Make persisting to the library opt-in.

https://developer.apple.com/documentation/shazamkit


