# Automations
Allow you to run shortcuts automatically subject to certain conditions.
# Automation suggestions
Centered around daily routines like going to and from work.  In iOS 14, we allow all actions like `NSUserActivity`, custom Intents, etc.

The system will learn and make suggestions from donations.

Tap 'Add automation' to add it to my collection.  
Donating is the only thing needed for the system to learn from these suggestions

```swift
import Intent

let intent = PlaceOrderIntent()
let soup = order.soup
intent.soup = INObject(identifier: soup.identifier.uuidString, displayString: soup.name)

intent.setImage(INImage(named: "Chowder"), forParameterNamed: "soup")
intent.setImage(INImage(named: "Home"), forParameterNamed: "deliveryLocation")

let interaction = INInteraction(intent: intent, response: nil)
interaction.donate { error in
	//handle the error
}
```

* powered by donations
* Supports all intents, including custom intents and user activities
* Works best with intents, since they open in the background etc.

[[Introduction to Siri Shortcuts - 18]]
[[Introducing Parameters for Shortcuts - 19]]

automation types

* time of day
* alarm
* apple watch workout
* arrive
* leave
* carplay
* airplane mode
* wifi
* bluetooth
* do not disturb
* low power mode
* battery level
* charger

iOS suggests various "daily routine" flows based on intent categories and correlating to different activities the user performs.
* `INPlayMediaIntent` for various
* `INStartWorkout` for at the gym
# Gallery and Editor

## "Shortcuts from Your Apps" section in the gallery
How are tehse curated?

* Suggest shortcuts using `INVoiceShortcutCenter`

```swift
import Intents

let suggestions = [INShortcut(intent: foo),
INShortcut(userActivity: bar)]

INVoiceShortcutCenter.shared.setShortcutSuggestions(suggestions)
```

Can also donate actions via `INInteraction` or `NSUserActivity`.

Actions at the bottom are suggested/donated.  May also display parameter options.
Actions at top are the ones marked configureable.  

We recommend using `INVoiceShortcutCenter` to set suggestions in addition to donating.

Adopt select system intents `INSendPaymentIntent`, `INRequestPaymentIntent`, `INRequestRideIntent`, these are automatically displayed

# next steups
* use `setShortcutSuggestions`
* donate your intents




