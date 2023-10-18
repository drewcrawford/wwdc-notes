#activitykit

# Live activity overview
Immersive, glanceable to way to keep track of an event.
discrete start and end, and provide real-time updates.  Or remotely with push notifications.

On iPhone 14 pro/max, we have the dynamic island.  Rendered with variable-width, compact presentation.

Dispalys up to 2 live activities at a time.  

At any time, a person can long-press the live activity to display the extended presentation, giving them even more glanceable information.  in the extended presentation, you can deeplink to different areas within your app, providing a rich user experience.

Some new experiences for live activities in iOS 17.  In addtion to the lock screena dn dynamic island, they appear in stanby.  And now iPad also supports.  Enable your implementation to bring your immersive activities to iPad.

* Leverage widget enhancements
* Add buttons or toggles

[[Bring widgets to life]]

* activityKit framework
* Programmatic layout with SwiftUI and WidgetKit
* Explict user action to beign a live activity.
* Permission model like notifications, user has to aauthorize
* must support lock screen and the dynamic island presentation
* Update remotely using push notifications

[[Update Live Activities with push notifications]]


# Lifecycle of live activities
* request
	* make sure your app is in the foreground, you have data, etc.
* update
* observe activity state
* end

```swift
import ActivityKit

struct AdventureAttributes: ActivityAttributes {
    let hero: EmojiRanger

    struct ContentState: Codable & Hashable {
        let currentHealthLevel: Double
        let eventDescription: String
    }
}
```

now let's set up the request

```swift
let adventure = AdventureAttributes(hero: hero)

let initialState = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "Adventure has begun!"
)
let content = ActivityContent(state: initialState, staleDate: nil, relevanceScore: 0.0)

let activity = try Activity.request(
    attributes: adventure,
    content: content,
    pushType: nil
)
```

staleDate - content is now out of date.  For now, I'll pass in nil.
relevanceScore -> determines the order in which each live activity appears when several are starting.  If i were going ot start another one, I might specify difference relevanceScore.  Default is 0.

pushType: indicates if live activity receives updates via push.  For this example, I'll set it to nil.

## Update


```swift
let heroName = activity.attributes.hero.name               
let contentState = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "\(heroName) has taken a critical hit!"
)

var alertConfig = AlertConfiguration(
    title: "\(heroName) has taken a critical hit!",
    body: "Open the app and use a potion to heal \(heroName)",
    sound: .default
)  
     
activity.update(
    ActivityContent<AdventureAttributes.ContentState>(
        state: contentState,
        staleDate: nil
    ),
    alertConfiguration: alertConfig
)
```

alert: iPhone, iPad, synced apple watch.  title/body only used on apple watch.  On iPhone/iPad, UI appears.


## Observe activity state
1.  Started'
2. finished
3. dismissed
4. stale
```swift
// Observe activity state asynchronously
func observeActivity(activity: Activity<AdventureAttributes>) {
    Task {
        for await activityState in activity.activityStateUpdates {
            if activityState == .dismissed {
                self.cleanUpDismissedActivity()
            }
        }
    }
}

// Observe activity state synchronously
let activityState = activity.activityState
if activityState == .dismissed {
    self.cleanUpDismissedActivity()
}
```

## End
```swift
let hero = activity.attributes.hero

let finalContent = AdventureAttributes.ContentState(
    currentHealthLevel: hero.healthLevel,
    eventDescription: "Adventure over! \(hero.name) has defeated the boss! Congrats!"
)

let dismissalPolicy: ActivityUIDismissalPolicy = .default

activity.end(
    ActivityContent(state: finalContent, staleDate: nil),
    dismissalPolicy: dismissalPolicy)
}
```

# Building live activity UI

```swift
import WidgetKit
import SwiftUI

@main
struct EmojiRangersWidgetBundle: WidgetBundle {
    var body: some Widget {
        EmojiRangerWidget()
        LeaderboardWidget()
        AdventureActivityConfiguration()
    }
}
```

needs to return some configuration


```swift
struct AdventureActivityConfiguration: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: AdventureAttributes.self) { context in
            AdventureLiveActivityView(
                hero: context.attributes.hero,
                isStale: context.isStale,
                contentState: context.state
            )
            .activityBackgroundTint(Color.navyBlue)
        } dynamicIsland: { context in
            // ...
        }
    }
}
```

first closure: specifies the lockscreen UI.  As my view context changes, this UI will be rendered for each update.  Let the system determine the appropriate dimensions.

compact presentation: leading and trailing.  Choose essential content in leading and trailing space, since the space is limited.  Users should be able to identify the specific activity by looking at the content.

```swift
struct AdventureActivityConfiguration: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: AdventureAttributes.self) { context in
            // ...
        } dynamicIsland: { context in
            DynamicIsland {
                // ...
            } compactLeading: {
                Avatar(hero: context.attributes.hero)
            } compactTrailing: {
                ProgressView(value: context.state.currentHealthLevel) {
                    Text("\(Int(context.state.currentHealthLevel * 100))")
                }
                .progressViewStyle(.circular)
                .tint(context.state.currentHealthLevel <= 0.2 ? Color.red : Color.green)
            } minimal: {
                // ...
            }
        }
    }
}
```

minimal view

```swift
struct AdventureActivityConfiguration: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: AdventureAttributes.self) { context in
            // ...
        } dynamicIsland: { context in
            DynamicIsland {
                // ...
            } compactLeading: {
                // ...
            } compactTrailing: {
                // ...
            } minimal: {
                ProgressView(value: context.state.currentHealthLevel) {
                    Avatar(hero: context.attributes.hero)
                }
                .progressViewStyle(.circular)
                .tint(context.state.currentHealthLevel <= 0.2 ? Color.red : Color.green)
            }
        }
    }
}
```
System displays the content in an expanded presentation.  For the expanded presentation, system divides it into different areas.



```swift
struct AdventureActivityConfiguration: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: AdventureAttributes.self) { context in
            // ...
        } dynamicIsland: { context in
            DynamicIsland {
                // Leading region
                DynamicIslandExpandedRegion(.leading) {
                    LiveActivityAvatarView(hero: hero)
                }

                // Expanded region
                DynamicIslandExpandedRegion(.trailing) {
                    StatsView(hero: hero, isStale: isStale)
                }

                // Bottom region
                DynamicIslandExpandedRegion(.bottom) {
                    HealthBar(currentHealthLevel: contentState.currentHealthLevel)
                    EventDescriptionView(hero: hero, contentState: contentState)
                }
            } compactLeading: {
                // ...
            } compactTrailing: {
                // ...
            } minimal: {
                // ...
            }
        }
    }
}
```

* show most essential content only
* simple design
* show additional details int he application

# Wrap up
* display live and glanceable information
* create interactive experiences
* engage with people on iOS and iPadOS
# Resources
* https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/live-activities
* https://developer.apple.com/documentation/ActivityKit/updating-and-ending-your-live-activity-with-activitykit-push-notifications
* https://developer.apple.com/documentation/ActivityKit/displaying-live-data-with-live-activities
* https://developer.apple.com/documentation/ActivityKit
* https://developer.apple.com/documentation/WidgetKit

[[Bring widgets to life]]
[[Update Live Activities with push notifications]]
[[Design dynamic Live Activities]]