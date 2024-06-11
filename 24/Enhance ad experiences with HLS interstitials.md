Explore how HLS Interstitials can help you seamlessly insert advertisements into your HLS content. We'll also show you how to use integrated timeline to tune your UI experience and build SharePlay for interstitials.

### Create a point interstitial event - 3:55
```swift
// Creating a single point interstitial event
let pointEvent = AVPlayerInterstitialEvent(primaryItem: playerItem, time: ten)
pointEvent.timelineOccupancy = .singlePoint
```

### Create a fill interstitial event - 4:07
```swift
// Creating a fill interstitial event
let fillEvent = AVPlayerInterstitialEvent(primaryItem: playerItem, time: ten)
fillEvent.timelineOccupancy = .fill
fillEvent.plannedDuration = CMTime(value: 15, timescale: 1)
```

### Create fill interstitial supplementing primary - 4:32
```swift
// Creating a fill interstitial event supplementing primary
let fillEvent2 = AVPlayerInterstitialEvent(primaryItem: playerItem, time: ten)
fillEvent2.supplementsPrimaryContent = true
fillEvent2.timelineOccupancy = .fill
fillEvent2.plannedDuration = CMTime(value: 15, timescale: 1)
```

### Draw simple transport bar with integrated timeline - 7:14
```swift
// Create AVPlayerItem and obtain its integrated timeline
let item = AVPlayerItem(url: ...)
let integratedTimeline = item.integratedTimeline

// Any time we need a new representation of the timeline state, we can request for a snapshot
let snapshot = integratedTimeline.currentSnapshot

// Using our snapshot, we can build a simple transport bar
drawUISlider(start: .zero, duration: snapshot.duration, currentPosition: snapshot.currentTime)

// Draw single-point interstitials on the transport bar
let pointSegments = snapshot.segments.filter { segment in
    segment.segmentType == .interstitial && segment.interstitialEvent?.timelineOccupancy == .singlePoint
}
for segment in pointSegments {
    drawPoint(position: segment.timeMapping.target.start)
}

// Draw range interstitials on the transport bar
let highlightFillSegments = snapshot.segments.filter { segment in
    if (segment.segmentType == .interstitial) {
        if let interstitialEvent = segment.interstitialEvent {
            return interstitialEvent.timelineOccupancy == .fill && !interstitialEvent.supplementsPrimaryContent
        }
    }
    return false
}
for segment in highlightFillSegments {
    let range = segment.timeMapping.target
    highlightRegion(start: range.start, end: range.end)
}
```

### Listen to snapshotsOutOfSyncNotification to update our UI - 8:26
```swift
// Listen to integrated timeline notifications to update our logic
for _ in NotificationCenter.default.notifications(named: AVPlayerItemIntegratedTimeline.snapshotsOutOfSyncNotification, object: integratedTimeline) {
    let reason = _.userInfo![AVPlayerItemIntegratedTimeline.snapshotsOutOfSyncReasonKey] as! AVPlayerIntegratedTimelineSnapshotsOutOfSyncReason
    switch(reason) {
    case .segmentsChanged:
        redrawTransportBar(snapshot: integratedTimeline.currentSnapshot)
    case .currentSegmentChanged:
        updatePlayerControls(snapshot: integratedTimeline.currentSnapshot)
    }
}
```

### Set ContentMayVary to false for Interstitial SharePlay support - 11:42
```swift
// Set contentMayVary to false for SharePlay support
let event = AVPlayerInterstitialEvent(primaryItem: playerItem, time: ten)
event.contentMayVary = false
event.timelineOccupancy = .fill
event.plannedDuration = CMTime(value: 15, timescale: 1)
event.supplementsPrimaryContent = true
```
# Resources
* https://developer.apple.com/forums/topics/media-technologies?cid=vf-a-0010
* https://developer.apple.com/documentation/avfoundation/media_playback/providing_an_integrated_view_of_your_timeline_when_playing_hls_interstitials
* 