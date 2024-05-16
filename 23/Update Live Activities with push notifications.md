Discover how you can remotely update Live Activities in your app when you push content through Apple Push Notification service (APNs). We'll show you how to configure your first Live Activity push locally so you can quickly iterate on your implementation. Learn best practices for determining your push priority and configuring alerting updates, and explore how to further improve your Live Activities with relevance score and stale date. To get the most out of this session, you should be familiar with ActivityKit and Live Activities. Check out “Meet ActivityKit” for an introduction to Live Activities.

ActivityKit
WidgetKit and SwiftUI for UI

[[Meet ActivityKit]]

Server updates Live Activities
No app foreground runtime

# Preparations

ActivityKit obtains a push token from APNs.  Unique for each live activity you request.
App needs to send it to server before it can start sending push updates.

Whenever you need to update live activity, send token to APNs.
We send payload to device, to wake widget extension.

APNs connection, new `liveactivity` push type.  Only available to servers with token-based connections to APNs.

Refer to Sending notification requests to APNs docs.
See 'establishing a token-based connection to APNs' docs.

Under signing and capabilities, add push notifications capability.  needed for activitykit to request tokens.


### 3:53 - Enabling push updates
```swift
func startActivity(hero: EmojiRanger) throws {
    let adventure = AdventureAttributes(hero: hero)
    let initialState = AdventureAttributes.ContentState(
        currentHealthLevel: hero.healthLevel,
        eventDescription: "Adventure has begun!"
    )

    let activity = try Activity.request(
        attributes: adventure,
        content: .init(state: initialState, staleDate: nil),
        pushType: .token
    )

    Task {
        for await pushToken in activity.pushTokenUpdates {
            let pushTokenString = pushToken.reduce("") { $0 + String(format: "%02x", $1) }
            
            Logger().log("New push token: \(pushTokenString)")
            
            try await self.sendPushToken(hero: hero, pushTokenString: pushTokenString)
        }
    }
}
```

ActivityKit knows to request a push token.  App needs to send push token to server.  Access the push token synchronously.  Do not access it immediately after creation, it will be nil.  Requesting a push token is an asynchronous process.  Possible for the system to update the push token throughout the lifetime of the activity.

loop over `pushTokenUpdates` async sequence.

Use async loop here because it handles not just the first push token but also subsequent updates.
Convert to hex string and log to debug console.  Will come in handy during testing in the next session.  Send push token alongside other data required for your app.

Unique for each activity.  Keep track of them for each live activity.
App will get foreground runtime for new push tokens.


# First push update

To send the update, send an HTTP request to apns

* apns headers
* apns payload

headers.
1.  apns-push-type: liveactivity
2. apns-topic: `<BUNDLE_ID>.push-type.liveactivity`
3. `apns-priority: 5` or 10.  5 is low priority, 10 is high priority.

For the first apns payload, send one that consists of 3 fields.
1.  timestamp - time interval in seconds since 1970
2. event - update/end.
3. content-state.  JSON object that be decoded into your activity's content state type.

Use foundation's JSON encoder type to create/read.


### 6:54 - APNs push payload: Updating
```json
{
    "aps": {
        "timestamp": 1685952000,
        "event": "update",
        "content-state": {
            "currentHealthLevel": 0.941,
            "eventDescription": "Power Panda found a sword!"
        }
    }
}
```



### 7:37 - Printing content state JSON
```swift
let contentState = AdventureAttributes.ContentState(
    currentHealthLevel: 0.941,
    eventDescription: "Power Panda found a sword!"
)

let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

let json = try! encoder.encode(contentState)
Logger().log("\(String(data: json, encoding: .utf8)!)")
```

JSON output with camel cased key slooks just like what I expected.  Always decoded using a JSON decoder with default decoding strategies.  Do not set custom encoding strategies!  Your JSON will be mismatched.

### 9:18 - Terminal: Constructing an APNs request with curl
```bash
curl \
  --header "apns-topic: com.example.apple-samplecode.Emoji-Rangers.push-type.liveactivity" \
  --header "apns-push-type: liveactivity" \
  --header "apns-priority: 10" \
  --header "authorization: bearer $AUTHENTICATION_TOKEN" \
  --data '{
      "aps": {
          "timestamp": '$(date +%s)',
          "event": "update",
          "content-state": {
              "currentHealthLevel": 0.941,
              "eventDescription": "Power Panda found a sword!"
          }
      }
  }' \
  --http2 https://api.sandbox.push.apple.com/3/device/$ACTIVITY_PUSH_TOKEN
```

"Sending push notifications using command-line tools" docs.  Make sure you follow "send a push notification using a token"

Get push token from debug console
set as environment variable

For url, make sure you use http2.  REference the activity_push_token variable.

your live activity will be updated with the new content state.  You may see situations where your live activity didn't update.

## Debugging update failures


1.  Ensure curl command was successful
2. View device logs in Console app.  see `liveactivitiesd`, `apsd`, `chronosd` processes.


# Priority and alerts

Always consider using low priority.

* Opportunistic delivery
* Less time-sensitive updates
* No limit

However some updates require user's immediate attention.  High priority updates.

* Immediate delivery
* Time-sensitive updates
* Budget depending on device condition
* System throttles push updates, impacting UX.


'live activity updates frequent updates' feature.  Add a key to `NSSupportsLiveActivitiesFrequentUpdates=YES`.
Requires frequent high-priority updates
Get higher update budget
Can still get throttled

Users can disable frequent updates in settings.
Get feature status: `ActivityAuthorizationInfo().frequentPushesEnabled`

Catch the user's attention so they can promptly go into the app and use a healing potion.


### 14:21 - APNs push payload: Alerting
```json
{
    "aps": {
        "timestamp": 1685952000,
        "event": "update",
        "content-state": {
            "currentHealthLevel": 0.0,
            "eventDescription": "Power Panda has been knocked down!"
        },
        "alert": {
            "title": "Power Panda is knocked down!",
            "body": "Use a potion to heal Power Panda!",
            "sound": "default"
        }
    }
}
```

Emoji rangers has support for multiple languages.  Only sending alerts in english is not ideal.  Handling localization on server is tricky.  However,


### 14:56 - APNs push payload: Alert localization
```json
{
    "aps": {
        "timestamp": 1685952000,
        "event": "update",
        "content-state": {
            "currentHealthLevel": 0.0,
            "eventDescription": "Power Panda has been knocked down!"
        },
        "alert": {
            "title": {
                "loc-key": "%@ is knocked down!",
                "loc-args": ["Power Panda"]
            },
            "body": {
                "loc-key": "Use a potion to heal %@!",
                "loc-args": ["Power Panda"]
            },
            "sound": "HeroDown.mp4"
        }
    }
}
```

Use your app's localization files!

I need to add the sound files to my app's target.  Set the sound field of the alert object to my sound's filename.  Now my alert looks/sounds great.


### 15:25 - APNs push payload: Alert sound
```json
{
    "aps": {
        "timestamp": 1685952000,
        "event": "update",
        "content-state": {
            "currentHealthLevel": 0.0,
            "eventDescription": "Power Panda has been knocked down!"
        },
        "alert": {
            "title": {
                "loc-key": "%@ is knocked down!",
                "loc-args": ["Power Panda"]
            },
            "body": {
                "loc-key": "Use a potion to heal %@!",
                "loc-args": ["Power Panda"]
            },
            "sound": "HeroDown.mp4"
        }
    }
}
```
# Enhancements

Use `event:end` to dismiss the payload.  Custom dismissal-date.

### 15:52 - APNs push payload: Dismissal
```json
{
    "aps": {
        "timestamp": 1685952000,
        "event": "end",
        "dismissal-date": 1685959200,
        "content-state": {
            "currentHealthLevel": 0.23,
            "eventDescription": "Adventure over! Power Panda is taking a nap."
        }
    }
}
```

recall, time interval in seconds since 1970.

### 16:44 - APNs push payload: Stale date
```json
{
    "aps": {
        "timestamp": 1685952000,
        "event": "update",
        "stale-date": 1685959200,
        "content-state": {
            "currentHealthLevel": 0.79,
            "eventDescription": "Egghead is in the woods and lost connection."
        }
    }
}
```

Activity will just continue to display previous content state until dismissed.

Can set `stale-date` field.  System will use this date to decide when to render your stale view.  Can provide from SwiftUI.
### 16:54 - Displaying a stale Live Activity UI
```swift
struct AdventureActivityConfiguration: Widget {
    
    var body: some WidgetConfiguration {
        
        ActivityConfiguration(for: AdventureAttributes.self) { context in
            AdventureLiveActivityView(
                hero: context.attributes.hero,
                isStale: context.isStale,
                contentState: context.state
            )
            .activityBackgroundTint(Color.gameWidgetBackground)
        }  dynamicIsland: { context in
            // ...
        }
        
    }
    
}
```

When there are multiple adventure activities simulaneously, order them on the lockscreen.  Best on the top, or dynamic island, use relevance score

### 17:19 - APNs push payload: Relevance score
```json
{
    "aps": {
        "timestamp": 16859q52000,
        "event": "update",
        "relevance-score": 100,
        "content-state": {
            "currentHealthLevel": 0.941,
            "eventDescription": "Power Panda found a sword!"
        }
    }
}
```


# Wrap up
* add support for push updates
* Test sending pushes from your terminal
* Add end-to-end support on server
* Consider priorities and alerts

[[Meet ActivityKit]]
[[Bring widgets to life]]


# Resources
* https://developer.apple.com/documentation/ActivityKit
* https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/establishing_a_token-based_connection_to_apns
* https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/live-activities
* https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns
* https://developer.apple.com/documentation/usernotifications/sending_push_notifications_using_command-line_tools
* https://developer.apple.com/documentation/ActivityKit/updating-live-activities-with-activitykit-push-notifications
* 