Learn how you can upgrade your app's AirPlay audio experience to be more robust and responsive. We'll show you how to adopt enhanced audio buffering with AVQueuePlayer, explore alternatives when building a custom player in your app, and share best practices.
# AirPlay overview
share videos, photos, music, and more.  With so many compatible devices, airplay is more popular than ever.  You can use iarplay in the following ways.

* audio - share your favorite music/podcasts by streaming to homepod or compatible speakers
* video - share from apple devices to apple tv or AP-compatible tvs
* airplay mirroring - share what's on your apple device, etc. to apple tv or airplay-compatible smart tv.

# Enhanced audio buffering

one or more devices.  Full ecosystem of devices.
* apple devices
* AV products from the world's top brands

to take airplay to the next level, we have EAB.  Even better home theater and multichannel experience

* whole home audio
* robust - faster than realtime speed
* responsive
* multichannel / dolby atmos
* intelligent use of apple lossless
* HLS interstitials

[[Explore AirPlay with Interstitials]]

I'm streaming audio from phone to homepod.  demo with streaming and turning off wifi.


# Add support to your app
identify the audio type.

set your audio session's category to playback.  Ensurs app continues to play while in background.**

Recommended to use the right audio type where possible.

'long form audio' - anything other than system sounds.

this year, we use on-device intelligence to learn your airplay preferences.


### Set the audio type - 4:00
```swift
let audioSession = AVAudioSession.sharedInstance()
try audioSession.setCategory(.playback, mode: .default, policy: .longFormAudio)
```

add new key value to info.plist `AVInitialRouteSharingPolicy = LongFormAudio`.  in xcode this is called 'AirPlay optmization policy'.  iOS will handle the rest and use on-devicel learning to suggest airplay routes.

* add an airplay picker
* add a media player
	* MPNowPlayingInfoCenter and MPRemoteCommandCenter

supporting EAB
* AvPlayer / AVQueuePlayer
* or AVSampleBufferAudioRenderer and AVSampleBufferRenderSynchronizer

both APIs work for non-airplay playback, including local or bluetooth.  Some develoeprs may want different APIs for airplay vs non-airplay playback. Your app can register for route change notification and act accordingly.

## AVPlayer
simplest way to support EAB.  Most app developers use AVQueuePlayer.  Handles most of what playback needs, including items, playback, seeking, etc.  Most of apple's own media apps use this.
### AVQueuePlayer - 7:23
```swift
let player = AVQueuePlayer()

let url = URL(string: "http://www.examplecontenturl.com")
let asset = AVAsset(url: url)
let item = AVPlayItem(asset: asset)

player.insert(item, after: nil)
player.play()
```

It's that simple.  You might be thinking: if it's just inserting a play item into the player, where's the AP part?  By using AVPlayer / AVQueuePlayer, you automatically get EAB for airplay.  For more avplayer functionality, see docs

If you have a unique app that does preprocessing or DRM we don't support, you can use AVSampleBufferAudioRenderer and friends.  
### Add the audio renderer to the render synchronizer - 8:28
```swift
let serializationQueue = DispatchQueue(label: "sample.buffer.player.serialization.queue")
let audioRenderer = AVSampleBufferAudioRenderer()
let renderSynchronizer = AVSampleBufferRenderSynchronizer()

renderSynchronizer.addRenderer(audioRenderer)
```

### Enqueue audio data - 8:50
```swift
serializationQueue.async { [weak self] in
    guard let self = self else { return }
    // Start processing audio data and stop when there's no more data.
    self.audioRenderer.requestMediaDataWhenReady(on: serializationQueue) { [weak self] in
        guard let self = self else { return }
        while self.audioRenderer.isReadyForMoreMediaData {
            let sampleBuffer = self.nextSampleBuffer() // Returns nil at end of data.
            if let sampleBuffer = sampleBuffer {
                self.audioRenderer.enqueue(sampleBuffer)
            } else {
                // Tell the renderer to stop requesting audio data.
                audioRenderer.stopRequestingMediaData()
            }
        }
    }

    // Start playback at the natural rate of the media.
    self.renderSynchronizer.rate = 1.0
}
```

for more details, see docs, where we describe this in-depth and have sample code.

CarPlay enhanced audio buffering
* car manufacture support
* wireless robust & responsive playback
* same APIs support CarPlay

[[Optimize CarPlay for vehicle systems]]

# Next steps
* configure audio sesison
* add an airplay picker
* adopt media player integration
* adopt AVqueuePlayer

* https://developer.apple.com/documentation/avfaudio/audio_engine/playing_custom_audio_with_your_own_player
* https://developer.apple.com/documentation/avfoundation/avqueueplayer
