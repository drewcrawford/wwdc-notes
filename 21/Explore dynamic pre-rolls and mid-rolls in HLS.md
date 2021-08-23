#hls 

Your users might prefer free ad-supported content over ad-free subscription content.  So we're talking about ads.

# Existing HLS ad insertion approaches
* Server side ad insertion with `#EXT-X-DISCONTINUITY` tags
* Static
* Transitions at segment boundary
* Ads are conditioned to match content encoding
* Significant back-end bookkeeping during playback

Client-side ad insertion

2-player scheme?  Coordinate playback between 2 players?

* Prefetching ad impacts ABR performance of the primary
* Airplay and PiP are difficult to do well

# HLS interstitial support
* Ads are separate assets scheduled on a primary program timeline
* Dynamic - late binding and rebinding to ad inventory
* Arbitrary placement of interstitials
* Offers built-in navigation restrictions
* Supports AirPlay and Picture-In-Picture
* Optimal use of network and system resources

# Playback flow - VOD
1.  Playback start
2.  Pause primary playback, play ad1
3.  play ad2
4.  Primary vod resumes from where it left off

We handle buffering across these streams.

For live, we rejoin main content after the duration, so we stay in sync.

Resume offset of 0 means "where you left off"

`X-RESUME-OFFSET`. Can be used to skip over in-stream ads.

Multiple ads with date-range tags.

# X-ASSET-LIST
List of interstitials.  Fetched at buffering time.

`X-PLAYOUT-LIMIT` can do an early return.

# X-RESTRICT
Navigation restrictions.  Prevent user from seeking after the ad.  
JUMP prevents you from seeking before the ad to after the ad.
SKIP prevents from playing the ad at a different rate.

Enforced by UI, tvOS enforced by AVKit.
Otherwise, it's up to you to enforce these restrictions

# Client ad reporting
`AVPlayerInterstitialEventMonitor` Nofies the client when an intersititial is scheduled or playing
`AVPlayerInterstitialEvent` Contains timing information for placing interstitials on a timeline

Event monitor
1.  Primary player
2.  interstitialPlayer
3.  events
4.  currentEvent
5.  eventsDidChangeNotification
6.  currentEventDidChangeNotification

Interstitial event has similar attributes.
1.  primaryItem
2.  identifier
3.  timeline
4.  date
5.  templateItems
6.  restrictions
7.  resumptionOffset
8.  playoutLImit
9.  userDefinedAttributes

use rdefined attributes from playlist can be surfaced to the application.  e.g. beacon URL, custom attributes, etc.

```swift
// Client observes server-side interstitial playback
let player = AVPlayer(url: movieURL) // movieURL has EXT-X-DATERANGE ad tags
let observer = AVPlayerInterstitialEventMonitor(primaryPlayer: player)
NotificationCenter.default.addObserver(
  forName: AVPlayerInterstitialEventMonitor.currentEventDidChangeNotification,
  object: observer,
  queue: OperationQueue.main) {
      notification_ in
      self.updateUI(observer.currentEvent, observer.interstitialPlayer)
}
```

# Client-side scheduling
* Programmatically set events

AVPlayerInterstitialEventController

while `events` is read-write on the monitor, it's read-write on the controller.

```swift
// Client inserted events

// Client schedules an ad pod at 10s into primary asset
let player  = AVPlayer(url: movieURL)  // no ads in primary asset
let controller = AVPlayerInterstitialEventController(primaryPlayer: player)
let adPodTemplates = [AVPlayerItem(url: ad1URL), AVPlayerItem(url: ad2URL)]
let event = AVPlayerInterstitialEvent( 
  primaryItem: player.currentItem,
  time: CMTime(seconds: 10, preferredTimescale: 1),
  templateItems: adPodTemplates,
  restrictions: [],
  resumptionOffset: .zero,
  playoutLimit: .invalid)

 controller.events = [event]
 player.currentItem.translatesPlayerInterstitialEvents = true
 let vc = AVPlayerViewController()
 vc.player = player
 player.play()
 ```
 
 # Wrap up
 Schedule ads using `EXT-X-DATERANGE`
 * `X_RESUME_OFFSET` 0 for VOD, skip for live
 * `X-ASSET-LIST` for late binding
 * `X-PLAYOUT-LIMIT` for early return
 * Navigation restrictions with `X-RESTRICT`
 * `AVPlayerInterstitialEventMonitor` to monitor ad playback
 * `AVPlayerInterstitialEventController` to schedule client side ads


* https://developer.apple.com/streaming/GettingStartedWithHLSInterstitials.pdf
* 

