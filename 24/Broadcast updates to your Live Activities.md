With broadcast push notifications, your app can send updates to thousands of Live Activities with a single request. We'll discover how broadcast push notifications work between an app, a server, and the Apple Push Notification service, then we'll walk through best practices for this capability and how to implement it.
### Subscribe a Live Activity to broadcast push notification updates - 7:50
```swift
// Request a Live Activity and subscribe to broadcast push notifications 
import ActivityKit 

func startLiveActivity(channelId: String) { 
    let gameAttributes = GameAttributes() 
    let initialState = GameAttributes.ContentState( home: 0, away: 0, update: "First Half" ) 
    
    try Activity.request( attributes: gameAttributes, content: .init(state: initialState, staleDate: nil), pushType: .channel(channelId) ) 
}
```