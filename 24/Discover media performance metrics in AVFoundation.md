Discover how you can monitor, analyze, and improve user experience with the new media performance APIs. Explore how to monitor AVPlayer performance for HLS assets using different AVMetricEvents, and learn how to use these metrics to understand and triage player performance issues.

### AVMetric Publishers - 0:01
```swift
public protocol AVMetricEventStreamPublisher { 
    func metrics<MetricType: AVMetricEvent>(forType metricType: MetricType.Type) -> AVMetrics<MetricType> 
    func allMetrics() -> AVMetrics<AVMetricEvent> 
} 

extension AVPlayerItem : AVMetricEventStreamPublisher
```

### Example showing how to obtain likely to keep up and summary metrics from AVPlayerItem - Swift - 0:02
```swift
let playerItem : AVPlayerItem = ... 
let ltkuMetrics = item.metrics(forType: AVMetricPlayerItemLikelyToKeepUpEvent.self) 
let summaryMetrics = item.metrics(forType: AVMetricPlayerItemPlaybackSummaryEvent.self) 

for await (metricEvent, publisher) in ltkuMetrics.chronologicalMerge(with: summaryMetrics) { 
    // send metricEvent to server 
}
```

### Example showing how to obtain likely to keep up and summary metrics from AVPlayerItem - Objective-C - 0:03
```objective-c
AVPlayerItem *item = ... 
AVMetricEventStream *eventStream = [AVMetricEventStream eventStream]; 
id<AVMetricEventStreamSubscriber> subscriber = [[MyMetricSubscriber alloc] init]; 

[eventStream setSubscriber:subscriber queue:mySerialQueue] 
[eventStream subscribeToMetricEvent:[AVMetricPlayerItemLikelyToKeepUpEvent class]]; 
[eventStream subscribeToMetricEvent:[AVMetricPlayerItemPlaybackSummaryEvent class]]; 
[eventStream addPublisher:item];
```