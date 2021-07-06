# Widget intelligence overview
Smart rotate.  Allows the system to automatically scroll to a widget in a stack.  
Use relevance signals from the widget.
Use behavioral patterns learned from app usage

Widget suggestions.  New way to discover your widget.

## Suggestions
* Inserts a new widget into a Smart Stack
* Uses relevance signals from the app
* Use behavioral patterns learned from app usage


# Donating to the system
Provide timely, glanceable information with obvious value to the user.

3 separate ways.

* `INRelevantShortcut`
* `TimelineEntryRelevance`
* `INInteraction`

|                        | Smart rotate | Widget suggestions | Adoption site    |
|------------------------|--------------|--------------------|------------------|
| INRelevantShortcut     |              | yes                | app              |
| TimelineEntryRelevance | yes          |                    | Widget extension |
| INInteraction          | yes          | yes                | app              |

## INRelevantShortcut
Donating this allows the system to insert the widget into a stack.

Specifies time-based or behavioral relevance

Supports both static and intent-configured widgets

### Static
* Create an `INRelevantShortcut`
* Set the `widgetKind`
* Set array of `INRelevanceProvider`
* Create the intent and set necessary parameters
* Create an `INRelevantShortcut` with the intent
* Set the widget's `widgetKind`
* Set array of `INRelevanceProviders`

### Dynamic?
* Set the array of relevant shortcuts in the default store
* Update donated relevant shortcuts by donating a new array
* Invalidate a relevant shortcut by omitting it from a newly donated array
* Donations with an intent can surface the intent on the Siri watch face

### `INRelevanceProvider`
Use `INDAteRelevanceProvider` for known time periods of relevance
Use no providers to tell the system to suggest based on behavioral patterns

Other relevance providers support only the Siri watch face and are **not supported by widget suggestions**.

In order to allow widget suggestions, we ensure our intent supports donations in definition file.

```swift
// Donate INRelevantShortcut for Widget Suggestions in app
// User has just made a purchase

var relevantShortcuts: [INRelevantShortcut] = []

let intent = ViewRecentPurchasesIntent()
intent.card = Card(identifier: card.identifier)
intent.category = .all

if let shortcut = INShortcut(intent: intent) {
    let relevantShortcut = INRelevantShortcut(shortcut: shortcut)
    relevantShortcut.shortcutRole = .information
    relevantShortcut.widgetKind = “CardRecentPurchasesWidget”

    let dateProvider = INDateRelevanceProvider(start: Date(), 
                                               end: Date(timeIntervalSinceNow: 1800))
    relevantShortcut.relevanceProviders = [dateProvider]

    relevantShortcuts.append(relevantShortcut)
}

INRelevantShortcutStore.default.setRelevantShortcuts(relevantShortcuts) { (error) in
    if let error = error {
        print("Failed to set relevant shortcuts. \(error))")
    } else {
        print("Relevant shortcuts set.")
    }
}
```

## TimelineEntryRelevance
* Indicatoin on a widget timeline entry about its worthiness of rotation
* Relative to all other timeline entries provided by the widget
* Score indicate worthiness of rotation
* Positive scores are eligible for Smart Rotate
* Score of zero is not eligible for Smart Rotate
* Scores are relative and their scaling should be consistent.

Duration specifies how long the score is valid
After duration passes, score is treated as zero
Duration of zero maintains score until next entry

```swift
// Appending TimelineEntryRelevance to a TimelineEntry in widget extension for Smart Rotate

struct CardRecentPurchasesEntry: TimelineEntry {
    let date: Date
    let relevance: TimelineEntryRelevance?
    let card: IntentCard?
    let category: PurchaseCategory
}

let relevance = TimelineEntryRelevance(score: 16.29, duration: 1800)
let entry = CardRecentPurchasesEntry(date: Date(), relevance: relevance, card: card,
                                     category: category)
```

Score based on price?

Categorize basedon price?
 
 Only allow the system to rotate our widget in 30 minutes after the purchase was made.  Can use duration.  Entries will be eligible for 30minutes, ineligible afterward.
 
 Really, timeline relevance scores can be whatever you like.  Can use which credit card, purchase, etc.  No one right way to do it.
 
 ## `INInteraction`
 Enable both smart rotation and widget suggestions through INInteraction.
 
 Every time the user views finromation in the app.
 
 Each donation is a datapoint that learns when the user tends to view information in the app.
 
 Predicted by the system using user's patterns
 enables suggestions for a configured widget
 System produces smart rotations and widget suggestions based on the prediction.
 
 Create a supported combination with necessary parameters to configure widget.
 
 Design suggestion UI when donation shows up elsewhere.
 
 ```swift
 // Donate INIntent in a card's purchases list in the app

.onAppear {
    let intent = ViewRecentPurchasesIntent()
    intent.card = Card(identifier: card.id.uuidString, displayString: card.name)
    intent.category = .all

    let interaction = INInteraction(intent: intent, response: nil)
    interaction.donate { error in
        if let error = error {
            print(error.localizedDescription)
        }
    }
}
```

Donations can appear on lock screen, spotlight, and siri suggestions widget.  Even if your widget doesn't adopt intents

[[Donate intents and expand your app's presence]]

Development, consider WidgetKit Developer Mode.

# Wrap up
* Widget suggestions
* Smart rotate

https://developer.apple.com/documentation/WidgetKit/TimelineEntry
https://developer.apple.com/documentation/WidgetKit/Making-a-Configurable-Widget


