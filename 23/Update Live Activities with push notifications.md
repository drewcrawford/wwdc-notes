Discover how you can remotely update Live Activities in your app when you push content through Apple Push Notification service (APNs). We'll show you how to configure your first Live Activity push locally so you can quickly iterate on your implementation. Learn best practices for determining your push priority and configuring alerting updates, and explore how to further improve your Live Activities with relevance score and stale date. To get the most out of this session, you should be familiar with ActivityKit and Live Activities. Check out “Meet ActivityKit” for an introduction to Live Activities.

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
# Resources
* https://developer.apple.com/documentation/ActivityKit
* https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/establishing_a_token-based_connection_to_apns
* https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/live-activities
* https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns
* https://developer.apple.com/documentation/usernotifications/sending_push_notifications_using_command-line_tools
* https://developer.apple.com/documentation/ActivityKit/updating-live-activities-with-activitykit-push-notifications
* 