#hls 

In 2021, we introduced HLS interstitials.  Really simple way to do ads.

This year, we've added feaguers that let you better optimize and fine-tune your presentation.


# Overview
separate ads can be directly referenced on the multivariant playlist.  Point to the multivariant playlist from primary content.

maybe your contente has some ads stitched in.  Can even replace these ads by stitching new ones overtop and specifying resume-duration.

Can schedule early return in live scenarios, etc.

[[Explore dynamic pre-rolls and mid-rolls in HLS]]

# Cueing ads
Force placement to beginning or end of the playback.  Using CUE.

PRE
* ad plays before start of program content
* schedule premium ad spots during live join

POST
* ad plays after program content
* schedule ads/interstitials at the end of event streams

ONCE
* ad plays only once
* use for rating screens, etc.

example.
# Prevent drift with SNAP
packager has one clock, encoder has another clock.  

If your packager clock is faster than encoder clock... you might miss some of the primary content hthat follows.

Can now use SNAP OUT to snap out at the segment boundary that is closest.  SNAP IN to snap back in the primary content that is closest to ad return time.

Use these only for live scnearios as clock drift shouldn't exist with pre-packaged video on demand.


# Query parameters
HLS_start_offset query parameter.  Supplied with the request for the interstitia'ls' asset list URL
* offset into asset list during live join
* for vod, offset into asset-list when seeking into a region replaced by the interstitial.

You might need a way to track the same playback session to avoid serving someone the same ad over and over.

`_HLS_primary_id` query parameter.  `X-Playback-SESSION-ID` if client supports it.
Set same `_HLS_primary_id` for primary and interstitial asset requests


# AVFoundation APIs
We've added support for CUE and SNAP here for client-side ads.  Can schedule as  a preroll or postroll, etc.

snapOut => startWithPrimarySegmentBoundary
snapIn => similar

AVPlayerInterstitialEvent object is now mutable.  Can create with the primary item and start time of the event, and then schedule the properties afterward.

```swift
// Client schedules an ad pod at 10s into primary asset
 let player  = AVPlayer( url: movieURL )  // no ads in primary asset
 let controller = AVPlayerInterstitialEventController( primaryPlayer: player )
 let adPodTemplates = [AVPlayerItem( url: ad1URL ), AVPlayerItem( url: ad2URL )]
 let event = AVPlayerInterstitialEvent( primaryItem: player.currentItem,
                              time: CMTime( seconds: 10, preferredTimescale: 1 ),
                                      )
 event.templateItems = adPodTemplates
 event.identifier = "Ad1"
 event.restrictions = []
 event.resumptionOffset = .zero
 event.playoutLimit = .invalid
 event.cue = .none
 
 controller.events = [event]
 player.currentItem.translatesPlayerInterstitialEvents = true
 let vc = AVPlayerViewController()
 vc.player = player
 player.play()
```

once the event is set on the controller, no changes will take effect as controller makes a copy.  You'd have to set the event once again on the controller

# Wrap up
* schedule the ads as pre/post rolls using `X-CUE`
* Play ad once by setting `X-CUE="Once"`
* `X-SNAP` to manage clock drift in live
* `_HLS_Start_offset` to optimize ads during live join
* `_HLS_primary_Id` for session management
* Automatic support in SharePlay
[[Display ads and other interstitials in SharePlay]]

* https://developer.apple.com/forums/tags/wwdc2022-10145
* https://developer.apple.com/forums/create/question?&tag1=125&tag2=449030
