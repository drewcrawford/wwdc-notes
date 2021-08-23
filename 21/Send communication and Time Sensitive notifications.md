New ways to enhance them, along with elevating certain categories.

# Visual updates
```swift
// Setting up notification actions with icons

let likeActionIcon = UNNotificationActionIcon(systemImageName: "hand.thumbsup")
let likeAction = UNNotificationAction(identifier: "like-action",
                                           title: "Like",
                                         options: [],
                                            icon: likeActionIcon)
        
let commentActionIcon = UNNotificationActionIcon(templateImageName: "text.bubble")
let commentAction = UNTextInputNotificationAction(identifier: "comment-action",
                                                       title: "Comment",
                                                     options: [],
                                                        icon: commentActionIcon,
                                        textInputButtonTitle: "Post",
                                        textInputPlaceholder: "Type here…")

let category = UNNotificationCategory(identifier: "update-actions",
                                         actions: [likeAction, commentAction],
                               intentIdentifiers: [], options: [])
```

Whena notification associated with this category is expanded, actions are presented.

Since notifications display app icon at larger size, provide new app icon size.

* app icon
* Content extensions
* Adopt action icons


# Notification management
There are new system controls which affect the delivery and interruption of notifications from applications.

* Notificaton summary
* Focus

## summary
Can now be delivered at scheduled times as a summary.  Reduces the number of active interruptions from incoming notifications and presents them collectively at set times.

All notifications delivered through the summary.

###  Best practices
* Include media attachments, these can be featured at the top
* Relevance score

## focus
Based on the activity or time of day.  Sleep, work, etc.  In such a configuration, device will filter presentation an dinterruption of notifications.

Selecting people, applications, that can send interruptive notifications.

* Control which applications and when
* Better interruption management
* Breakthrough if allowed
* Communication and Time Sensitive


# Interruptions
* Passive
	* No sound or vibrations
	* Silent
	* Do not require immediate attention
	* Should be seen eventually
	* Dining recommendations
	* New episode avilability
* Active
	* Will not break through system management of notifications
	* Default level
	* Do not interrupt if configured not to
	* Sports updates
	* Live stream video
* Time Sensitive
	* Alert like active notifications
	* Allowed to breakthrough such as notification summary and focus
	* Only use when relevant to actively interrupt requiring immediate attention
	* Account security
	* package delivery
* Critical
	* Equivalent to prior critical behavior
	* Breakthrough system controls
	* Bypass ringer switch
	* Continue to require entitlement
		* Severe weather
		* Local safety alerts

`UNNotificationInterruptionLevel`

```swift
// Interruption levels
// Local notification

import UserNotifications

let content = UNMutableNotificationContent()
content.title = "Passive"
content.body = "I’m a passive notification, so I won’t interrupt you."
content.interruptionLevel = .passive

let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)

let request = UNNotificationRequest(identifier: "passive-request-example",
                                       content: content,
                                       trigger: trigger)
```

## Announce
Siri can announce notifications if there are compatible devices.

To get this behavior, needed to use a category.
In iOS 15, the requirement is deprecated.  Now announce is supported for *any* notification.  Communication and time sensitive will be announced by default.

Carplay => communication can be configured to announce automatically.


# Time Sensitive
* Require immediate attention
* Breakthrough (notification summary/focus) *if allowed to do so*.
* Important to maintain trust when sending these.
	* Do not overuse their interruptive nature
	* only use when relevant
* User option to turn off
* Enable capability
* Set interruption level

```swift
// Time Sensitive
// Local notification

let content = UNMutableNotificationContent()
content.title = "Urgent"
content.body = "Your account requires attention."
content.interruptionLevel = .timeSensitive

let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 0, repeats: false)

let request = UNNotificationRequest(identifier: "time-sensitive—example",
                                       content: content,
                                       trigger: trigger)
```

Set interruption level in aps payload to `time-sensitive`.


# Communication

Prioritize people.

New API that allows your application to signal what notifications are communications and the associated people.

Siri suggests important people that should be allowed to interrupt.

As your user interacts with the device, taking calls, etc., siri learns who is a candidate to break through focus and notification summaries.

Prominent avatars.  Title/subtitle are standardized, always includes sender in the title and in the case of group communications, receipients in the subtitle.

Siri announces communications on supported devices including homepod, etc.

Siri provides suggestions to prioritize these.  

Siri shortcuts are available for tasks with those people.  Siri suggests relevant people to breakthrough in the focused configuration.

New API lets you add sirikit calls and message intents to notifications.  Intents are driven by common tasks, your app donates these by placing a call or receiving a message.

Integration with intents
* StartCallIntent
* SendMessageIntent

Content-providing profile is the mechanism to associate intent with notifications

```swift
// New UserNotifications API

@available(macOS 12.0, *)
public protocol UNNotificationContentProviding : NSObjectProtocol {}

open class UNNotificationContent : NSObject, NSCopying, NSMutableCopying, NSSecureCoding {
    // ...

    @available(macOS 12.0, *)
    open func updating(from provider: UNNotificationContentProviding) throws 
                                                    -> UNNotificationContent

    // ...
}
```

Need to update a notification content object with a sirikit intent in your notification service extension.  These are local to the device, so they need to be customized locally.

Main app process for local notifications.  In `didReceive`, update the content of a pushed payload with an intent.

* Donate before updating notification content
* Always set interaction to incoming
* Do not conform your own objects to content providing

* Enable capability
* Add intent types to `NSUserActivityTypes` in Info.plist

```swift
func sendMessage(...) {
    // ...

    let intent: INSendMessageIntent = // ...
    let interaction = INInteraction(intent: intent, response: nil)

    interaction.direction = .outgoing
    interaction.donate(completion: nil)
}
```

Siri uses the outgoing intents to aid in who should breakthrough.

Contact suggestions: siri only learns from outgoing intent donations.  So not cluttered by e.g. spam clalers

* Proper usage of communication intents
* Communication intents are comprised of people
* Provide rich, accurate representations of each person

INPerson

```swift
// Create INPerson

let person = INPerson(personHandle: handle,
                    nameComponents: nameComponents,
                       displayName: displayName,
                             image: image,
                 contactIdentifier: contactIdentifier,
                  customIdentifier: customIdentifier,
                           aliases: nil,
                    suggestionType: suggestionType)
```

```swift
// Create INSendMessageIntent
// In your notification service extension

let intent = INSendMessageIntent(recipients: [person2],
                        outgoingMessageType: .outgoingMessageText,
                                    content: content,
                         speakableGroupName: speakableGroupName,
                     conversationIdentifier: conversationIdentifier,
                                serviceName: serviceName,
                                     sender: person1,
                                attachments: nil)

let interaction = INInteraction(intent: intent, response: nil)
interaction.direction = .incoming
```

Each paramter you populate provides a more intelligent user experience.

## Next steps

* Implement SiriKit intents by starting with an intents extension
* Donate intents for both incomign and outgiong communication notifications
* Start donating incoming start calls as well
* Create a notification service extension
* Update notification content with the communication intent

# Wrap up
* Visual updates and management options
* Interruption levels
* Time Sensitive
* Communications

* https://developer.apple.com/documentation/sirikit/instartcallintent
* https://developer.apple.com/documentation/sirikit/insendmessageintent
* https://developer.apple.com/documentation/usernotifications
* https://developer.apple.com/documentation/sirikit


