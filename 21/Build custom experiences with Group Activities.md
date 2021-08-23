# Introduction
* Shared experiences across devices
* Focus on coordinated media playback

[[Coordinate media experiences with Group Activities]]

Demo.


# Activity creation
1.  Configure activity
2.  Activate activity

Only the configure part is different for custom activities vs media experience.

Activity should contain all information

```swift
struct DrawTogether: GroupActivity {

    var metadata: GroupActivityMetadata {
        var metadata = GroupActivityMetadata()
        metadata.title = NSLocalizedString("Draw Together",
                                           comment: "Title of group activity")
        metadata.type = .generic
        return metadata
    }

}
```

By setting type to `.generic`, we made this a custom activity.  Crucial for custom activity.

Project settings=> capabilities => group activities.  Need to add entitlement.

## Activate


# Session management
Send and receive custom data.  

[[Coordinate media experiences with Group Activities]]

You should be familiar with
1.  Receive session
2.  Prepare for playback
3.  Join session

Here step 2 is "configure session" instead.

## Receive session
```swift
.task {
	for await session in DrawTogether.sessions() {
		configureGroupSession()
	}
}
```

## Configure session
```swift
let messenger = GroupSessionMessenger(session: groupSession)

// 1. Define
struct UpsertStrokeMessage: Codable {
    let id: UUID
    let color: Color
    let point: CGPoint
}

// 2. Receive
for await (message, context) in messenger.messages(of: UpsertStrokeMessage.self) {
    // Handle message
}

// 3. Send
do {
    try await messenger.send(UpsertStrokeMessage(id: stroke.id, color: .red, point: point))
} catch {
    // Handle error
}
```

## Other considerations
* Reliable transport
* Message constraints
* Flow-control and rate-limiting
* Versioning

# Polish
## Late-joiners
* Depends on your app
	* Draw together - everyone needs the same drawing
	* Devices can observe `.activeParticipants` and resend data

only send capture message to new participants
Only receive messages that are newer (more points)

## Changing activity
* new session
* Updating activity

Preferred: new session
`.prepareForActivation()`.  Makes it easier to reason about a consistent state.

Gives a system an indication of a major change "which will be used to notify the user"

Updating activity.  Multiple songs playing after each other, etc.  `.activity = newActivity`.  From there you listen to that change.

## GroupState observation
When elgibile for a group session, we show a button

`.isEligibleForGroupSession` publisher.  Dynamically show/hide our button.

# Wrap up
* Activity creation
* Session management
* Polish

[[Design for Group Activities]]
[[Coordinate media experiences with Group Activities]]

* https://developer.apple.com/documentation/avfoundation/media_playback_and_selection/supporting_coordinated_media_playback
* https://developer.apple.com/documentation/GroupActivities

