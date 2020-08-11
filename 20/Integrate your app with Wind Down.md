# Wind Down
All of these shortcuts can be accessed from shortcuts in a new "sleep mode" collection.

Any existing or new shortcut can show up in this collection by flipping the switch in shortcuts detail view.
# Integrate your actions
Adopt one of the builtin intents, or define your own custom intents.

`shortcutAvailability` on the `INIntent` or `NSUserActivity`.

These options match the categories in wind down setup flow
* journaling
* mindfulness
* music
* podcasts
* Prepare for Tomorrow
* Reading
* Yoga and Stretching

```swift
let playSoundIntent = INPlayMediaIntent()
playSoundIntent.shortcutAvailability = .sleepMusic

let shortcut = INShortcut(intent: playSoundIntent)
let shortcutCenter = INVoiceShortcutCenter.shared
shortcutCenter.setShortcutSuggestions([shortcut])
```

Now my bedtime action will show in the music section.

Not every action will help people reach their sleep goals.



# Suggestions and donations

* Provide shorcuts with `INVoiceShortcutCenter`.  If your action schange, make sure to update
* Donate to power predictions when people do the actions.

How to decide between suggestions and donating?

## Feature your actions with `INVoiceShortcutCenter` -> ensures that actions are featured in wind down.  

```swift
import Intents

let playSoundIntent = INPlayMediaIntent()

playSoundIntent.shortcutAvailability = .sleepMusic
playSoundIntent.suggestedInvocationPhrase = "Play Soundscape of the Day"

let shortcut = INShortcut(intent: playSoundIntent)
INVoiceShortcutCenter.shared.setShortcutSuggestions([shortcut])
```

## Donate `INInteractions` or `NSUserActivities` for redictions

```swift
import Intents

let playSoundIntent = INPlayMediaIntent()

playSoundIntent.shortcutAvailability = .sleepMusic
playSoundIntent.suggestedInvocationPhrase = "Play Counting Sleepy Dinosaurs"

let interaction = INInteraction(intent: playSoundIntent, response: nil)
interaction.donate { error in
	//handle the error
}
```

Donating a user activity

```swift
import Intents
import UIKit

let userActivity = NSUserActivity(activityType: "com.bedtime.Bedtime.playSound")

userActivity.isEligibleForSearch = true
userActivity.isEligibleForPrediction = true

userActivity.title = "Play Running Water"
userActivity.suggestedInvocationPhrase = "Play Running Water"
userActivity.shortcutAvailability = .sleepMusic

viewController.userActivity = userActivity
```
## choosing a phrase
include the name of the track
Limit length of phrase by omitting unneccessary information
