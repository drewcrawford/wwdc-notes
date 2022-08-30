Media metadata publishing and playback interactions.

Number of places where media metadata appears.  Now playing.  Control center expanded now playing.  Lock screen.  Apple watch.  AVKit PlayerUI.  Control center tvos?  Notificationson tvOS.  tvOS inactive full-screen overlay.

With a growing number of devices, properly publishing now playing information adn responding to remote commands is now more important than ever
# MPNowPlayingSession
* Represents a playback session
* best way to publish metadata
* Can concist of multiple players
* Now available on iOS 16
* Offers manual control of Now Playing \* status if there's multipel active sessionsx
* Supports traditional manual metadata publishing and automatic publishing
* Receives playback controls from the ecosystem
* Not for use with `AVPlayerViewController` #avkit 

Now playing
* the app that will show up on lock screen, contro center, etc.
* The "main" media currently playing
* Multiple sesisons means app responsible for promoting/demoting sessions as "now playing"

Eligibility
* At least one remote command
* AVAudioSession that is not mixable
	* mixable is used for playing back notifications, etc.  Good indication that it isn't now playing.

if you support pip, you might have 2 sessions, one for main and one for playback.  


```swift
// playing Magnificent

self.session = MPNowPlayingSession(players: [player])



// Playing different WWDC sessions, one full screen and one in PiP

self.session = MPNowPlayingSession(players: [player])
self.pipSession = MPNowPlayingSession(players: [pipPlayer])



// playing multi-view race

self.session = MPNowPlayingSession(players: [topLeft, topRight, bottomLeft, bottomRight])
```

```swift
// Promoting and demoting sessions as Now Playing

self.session = MPNowPlayingSession(players: [player])
self.pipSession = MPNowPlayingSession(players: [pipPlayer])

// if the content in PiP is promoted to full screen, swap active

self.pipSession.becomeActiveIfPossible { becameActive in
    // if success, pipSession data populates lock screen, etc, and
    // controls from lock screen, etc are routed to pipSession
}
```

# Playback interactions
```swift
// Example of responding to play and pause commands

self.session = MPNowPlayingSession(players: [player])

// respond to play commands
self.session.remoteCommandCenter.playCommand.addTarget { event in
    player.play()
    return .success
}

// respond to pause commands
self.session.remoteCommandCenter.pauseCommand.addTarget { event in
    player.pause()
    return .success
}
```

Add handlers for every command your app supports.
```swift
// Example responding to skip forward commands

self.session.remoteCommandCenter.skipForwardCommand.preferredIntervals = [15.0]
self.session.remoteCommandCenter.skipForwardCommand.addTarget { event in
    let skipCommand = event as! MPSkipIntervalCommandEvent
    player.seek(to: CMTimeAdd(player.currentTime(), CMTimeMakeWithSeconds(skipCommand.interval, preferredTimescale: 1)))
    return .success
}

// commands can also be disabled. for example, during an ad:
self.session.remoteCommandCenter.skipForwardCommand.isEnabled = false

// add handlers for all commands that are applicable to the content // https://developer.apple.com/documentation/mediaplayer/mpremotecommandcenter
```

# Automatic metadata publishing
* Handles publishing metadata observably by MPNowPlayingSession directly
* duration
* elapsed time
* state (paused/oplaying)
* progress

other metadata can be added
* title, description, artwork, etc.
```swift
// Example of setting artwork metadata
let artwork = MPMediaItemArtwork(image: image)
let title = "Magnificent"

playerItem.nowPlayingInfo = [
    MPMediaItemPropertyTitle: title,
    MPMediaItemPropertyArtwork: artwork,
    // …
]

self.session = MPNowPlayingSession(players: [player])
self.session.automaticallyPublishNowPlayingInfo = true
```
suppose you want to leave out ads
```swift
// Example with ads that should not contribute to elapsed time and duration

let preroll = MPAdTimeRange(timeRange: CMTimeRange(start: CMTime.zero, duration: CMTimeMakeWithSeconds(30, preferredTimescale: 1)))


playerItem.nowPlayingInfo = [
    …
    MPNowPlayingInfoPropertyAdTimeRanges: [preroll]
    …
]

self.session = MPNowPlayingSession(players: [player])
self.session.automaticallyPublishNowPlayingInfo = true
```

# Publishing with AVKit
Similar to atuomatic publsihing with MPNowPlayingSession for tvOS
* metadata added directly to AVPlayerItem
* elapsed time, duration, etc. publisheid automatically
* Use MPNowPlayingSession with aVKit on iOS
  also populates player UI
## AVMetadataItem
* player item has `externalMetadata` array of `AVMetadataItems`
* identifier: key to indicate metadata type ex title, etc.
* Value: the value of the metadata item
* DataType: used for artwork.  ex, JPEG data.
* ExtendedLanguageTag: language for the value (usually "und")

avoid setting en-US for english content because ex spanish users won't see it at all

```swift
// Example of setting artwork metadata

let path = Bundle.main.path(forResource: "poster", ofType: "jpg")
let posterData = FileManager.default.contents(atPath: path!)!

let artwork = AVMutableMetadataItem()
artwork.identifier = .commonIdentifierArtwork
artwork.value = posterData as NSData
artwork.dataType = kCMMetadataBaseDataType_JPEG as String
artwork.extendedLanguageTag = "und"

let title = AVMutableMetadataItem()
title.identifier = .commonIdentifierTitle
title.value = "Magnificent" as NSString
title.extendedLanguageTag = "und"

playerItem.externalMetadata = [artwork, title]
```

Visible to users set to any language, we set `und`.

* commonIdentifierTitle
* commonIdentifierArtwork, etc.
* set as many as possible

# Manual metadata publishing
* manual control over low level playback state
* elapsed time, duration, playback rate, etc. not determined for you
* Must use `.shared()` instance of `MPRemoteCommandCenter` for remote commands.

```swift
// Example of setting Now Playing information

let artwork = MPMediaItemArtwork(image: image)

let nowPlayingInfo = [
    MPMediaItemPropertyTitle: title,
    MPMediaItemPropertyArtwork: artwork,
    MPMediaItemPropertyPlaybackDuration: playerItem.duration,
    MPNowPlayingInfoPropertyElapsedPlaybackTime: player.currentTime().seconds,
    MPNowPlayingInfoPropertyPlaybackRate: player.rate
]

MPNowPlayingInfoCenter.default().nowPlayingInfo = nowPlayingInfo
```

# Wrap up
* Maximize ecosystem integration for your users
* New integrations should take advantage of automatic publishing
* Existing integrations can switch to automatic publishing

```swift
// On any non-linear time change, playback rate change, or play/pause

var nowPlayingInfo = MPNowPlayingInfoCenter.default().nowPlayingInfo
nowPlayingInfo[MPNowPlayingInfoPropertyElapsedPlaybackTime] = player.currentTime().seconds
nowPlayingInfo[MPNowPlayingInfoPropertyPlaybackRate] = player.rate
```



* https://developer.apple.com/forums/tags/wwdc2022-110338
* https://developer.apple.com/forums/create/question?&tag1=307&tag2=516030
* https://developer.apple.com/documentation/mediaplayer
