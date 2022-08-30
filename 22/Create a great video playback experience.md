We build a completely new media player from the gruond up.  new look and feel to keep the focus on the content and fit within a broader spectrum of apps.

New interaction models.  

# Player redesign
Skip interval => 10s
Removing a slider knob marking the timeline's current position.  Instead it can be interacted with from anywhere along the slider.

Replaced the video aspect control with a more intuitive pinch-to-zoom gesture. 
Portrait content.

iPadOS => keyboards, trackpads, mics, etc.

New ways to interact with controls.  Change the video's fill aspect.  Now use a pinch to move through the available zoom level.  Pinching in => safe area.  Piinching out => fully fill.

Tap the center of the display even while the controls are hidden to play/pause the video.  New way to navigate through media timeline.  Scroll from anywhere over the video.

Title, subtitle, description.  Can provide strings for each from aVMetadtaItem.
```swift
// Setting content external metadata

let titleItem = AVMutableMetadataItem()
titleItem.identifier = .commonIdentifierTitle
titleItem.value = // Title string

let subtitleItem = AVMutableMetadataItem()
subtitleItem.identifier = .iTunesMetadataTrackSubTitle
subtitleItem.value = // Subtitle string

let infoItem = AVMutableMetadataItem()
infoItem.identifier = .commonIdentifierDescription
infoItem.value = // Descriptive info paragraph

playerItem.externalMetadata = [titleItem, subtitleItem, infoItem]
```

Titles should be short and descriptive.  Similarly, we added a subtitle by creating a metadata item.

Chevron, info panel.  Shows content description.  

# designing playback experiences
We took a step back from what we built in the past.  What makes for ag ood ux?  We wanted to share this process with you.

3 things that make media experience great
1.  Intuitive.  Easy, familiar, natural, etc.
2. Integrated.  app, system, etc.
3. Content forward.  At the end of the day, you're there for the media.

## Intuitive
Familiarity
* draw on past experiences
* Widely understood

Draw on
* platform paradigms
	* Animates from the bottom.  So it can be dimissed by pushing it back down.
* real world
	* scrolling gesture.  Similar to rolling a toy car across the table, each swipe has momentum.  Continuing the movement of the timeline past the direct interaction.
	* Builds an association with real-world moving objects.

## Integrated
All the functionality, features, and devices people expect to work, just work.  Importantly, they work in a way that's consistent with your expectations.

Control center, picture and picture, etc.  

Your app should feel like a native part of the system.
* CoreSpotlight integration
* Now playing info
* MediaRemote commands
* TV app integration

* AirPlay
* SharePlay
* PiP

Ensuring all remotes is critical.  We always recommend using the system media player on tvOS.

* TV remotes
* keyboard
* trackpad/mouse
* game controller
* headphone controls

building support for all of these integration points, features, and hardware ocnfigurations, can be daunting.  That's why we built AVPlayerViewController.

## Content forward
The primary goal of your design.  The defining aspect of a great media experience.

Provide content metadata
* title and subtitle
* description
* thumbnail
* season and episode information
* start and end dates for live streams

Don't letterbox.  Include support for latest media standards, hdr, dolby atmos, etc.
audio and subtitle tracks for multiple languages

Keep focus on your content.

# Visual intelligence
Live text on video frames.

`.allowsVideoFrameAnalysis`.  Enabled for all apps linking new SDKs.  
when to disable:
* Performance critical applications
* when interaction with the video is not expected, such as splash screens
* visual intelligence wtih AVPlayerLayer

[[Designing with Visual Intelligence]]
[[Use VisionKit for Live Text]]

# Interstitials
Until now, interstitials were onyl supported on tvOS.  Adding support to iOS.  When timeline hits a marker, we begin playing the interstitial.  You'll get this behavior automatically.

`AVInterstitialTimeRange` from tvOS.  `interstitialTimeRanges`.  When creating interstitial events locally, a time range will be synthesized for each event.

Unlike tvOS, this is read-only.  They either are defiend in the HLS stream or via interstitial events.  Similar to setting `translatesPlayerInterstitialEvents` to yes.

Beinging over delegate methods for interstitials.

```swift
// Creating a skip button for a preroll ad

let eventController = AVPlayerInterstitialEventController(primaryPlayer: mediaPlayer)

let event = AVPlayerInterstitialEvent(primaryItem: interstitialItem, time: .zero)
event.restrictions = [
	.requiresPlaybackAtPreferredRateForAdvancement,
	.constrainsSeekingForwardInPrimaryContent
]

eventController.events.append(event)


func playerViewController(playerViewController: AVPlayerViewController, willPresent interstitial: AVInterstitialTimeRange) {
	showSkipButton(afterTime: 5.0, onPress: {
		eventController.cancelCurrentEvent(withResumptionOffset: CMTime.zero)
	})
}
```

To learn more, 
* HLS stream based itnerstitials
* aVFoundation interstitials API

[[What's new in HLS Interstitials]]
[[Explore dynamic pre-rolls and mid-rolls in HLS]]

# Playback speed
Both AVPlayerView and AVPlayerVC can display these.  We're making tihs available on macOS, iOS, and tvOS.

All apps linking against these SDKs will get this fucntionality automatically with no additional changes required.  However, depending on your usecase, some applications may want to modify the list of speeds, select a speed, etc.

`AVPlaybackSpeed`.  User-selectable speed options.  3 properties
* rate
* localizedName.  example, a speed of 2.5 might use a name of "two and a half times speed"
* localizedNumericName: string displayed in the playback menu.  Use this speed in your UI.

AVPlayerVC etc.
* speed property defines a custom list of playback speeds.  By default, this will be set to AVPlaybackSpeed system speeds list.
* selectedSpeed
* selectSpeed()
	* Never implicitly override the chosen playback speed



```swift
// Setting custom playback speeds


let player = AVPlayerViewController()
player.player = // Some AVPlayer

present(player, animated: true)


let newSpeed = AVPlaybackSpeed(rate: 2.5, localizedName: "Two and a half times speed”)

player.speeds.append(newSpeed)
							   ```




```swift
// Hiding the playback speed menu


let player = AVPlayerViewController()
player.player = // Some AVPlayer

present(player, animated: true)


player.speeds = []
```

Always all AVPlayerPlay to begin playback.  Never start playback by calling setRate(1), since the selected rate might not be 1.
# Wrap up
* redesigned media player
* design great media experiences
* visual intelligence features
* new APIs

* https://developer.apple.com/forums/tags/wwdc2022-10147
* https://developer.apple.com/forums/create/question?&tag1=327&tag2=411030
* https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/video/
