#hls 

Would you like to offer your customers a seamless playback style?  Continuity?  Etc.

# Playback experiences
* Item contintuity
* Compilations albums and live
* Linear programming
* Dynamic scene stitching

**Apple music is delivered this way.**

# Media requirements
By providing in your HLS manifests new variants, you're enabling a gapless transition.

Match the following audio traits:
* DRM
* Codec
* Channel count
* Bit depth
* Sample rate

Offer the same set of variants for consecutively enqueued items.

Adhere to CMAF authoriing guidance.
Use an edit list 'elst' to
* Omit output of priming audio data \[7.5.12 and 10.4.5\]
* Omit output of remainder frames


# Playlist examples
Item 1 => Four variants.  pair of 720p, pair of 1080p.  Within each pair, I offer a variant encded with HE and LC.

Consider 720p with HE-AAC.  Will select similar variant on the subsequent item.  This is true even if network conditions support a higher-quality tier.  Gapless transitions are higher priority.

When playl proceeds, we can switchover.

Item 2 => If we didn't offer HE-AAC on the second item.  Unable to maintain gapless.

```swift
// create two items, enqueue in order, 
// and play gaplessly

let item1 = AVPlayerItem(url: url1)
let item2 = AVPlayerItem(url: url2)

let player = AVQueuePlayer()
	
player.insert(item1, after: nil)
player.insert(item2, after: item1)

player.play()
```

Note that our items source from different URLs.  Inform queue player of the intended sequence, press play.

## Advanced technique
* Common AVAsset
* Multiple AVItems
* In - `seekToTime`
* Out - `forwardPlaybackEndTime`

I achieve a customized dynamic sequence.
# Demo
I've left AVKit controls visible so you can see 3 distinct resources.

# Wrap up
* Provide variant alternates of matching audio format
* AVQueuePlayer with items in sequence

[[Explore HLS variants in AVFoundation]]

* https://developer.apple.com/streaming/



