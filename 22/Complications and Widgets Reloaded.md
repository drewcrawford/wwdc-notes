#widgetkit 

API additions to WidgetKit that enable you to write accessory widgets for the lock screen.  

Seek out prior WK sessions.  

# Complication timeline
Quick, glanceable information.  
* third-party complications
* Graphic complications
* swiftUI complications
* multiple complications

Today we reimagined them on watchOS and iOS.  Build great glanceable widgets and complications across both platforms.

New widget families with `accessory` prefix.  `accessoryRectangular`.
circular => brief info, gauges, progress views
inline => text only slot present on many faces.  Many sizes.
corner => specific to watchOS, small circle with gauges and text.  
These replace prior options.

[[Go further with WidgetKit Complications]]

# Colors
System controls look of accessory widgets.
* full color
	* exactly as you specified.  Many take on a colorful appearance.
* accented
	* system tints content ina  number of ways, some of which are inverted.  
* vibrant
	* Deaturated.  Lock screen background.  System maps you into greyscale.  Dark => less prominent blur.  Avoid using transparent colors in this mode.  Use darker colors or black.  

Accented:
```swift
VStack(alignment: .leading) {
    Text("Headline")
        .font(.headline)
        .widgetAccentable()
    Text("Body 1")
    Text("Body 2")
}.frame(maxWidth: .infinity, alignment: .leading)
```

## AccessoryWidgetBAckground:
```swift
ZStack {
     AccessoryWidgetBackground()
     VStack {
        Text("MON")
        Text("6")
         .font(.title)
    }
}
```

some styles can be enhanced with background.  Takes on different appearances and tuned by the system per style.

black in vibrant.  Soft transparent in others.


# Project setup
Adding widget extension to your project.  Exists on iOS< bruogh tto watchOS.  Many of you have widgets already.  Let's talk about adidng new widgets and complications.

Keeps you up to date with health and recharge time.  We've brought this to watchOS, bringing our favorite app to our wrists.  Today we extend.

Add new watchOS target.  Duplicate existing target, rename, change bundle identifier to be prefixed with the watch apps.
Base SDK => watchOS.  Embed our new extension in our watch app.

Get our code building on watchOS.  Becuase system families are unavailabe on watch, specify supported families with platform macros.

Add previews for the new families.

```swift
EmojiRangerWidgetEntryView(entry: SimpleEntry(date: Date(), relevance: nil, character: .spouty))
                .previewContext(WidgetPreviewContext(family: .accessoryCircular))
                .previewDisplayName("Circular")
            EmojiRangerWidgetEntryView(entry: SimpleEntry(date: Date(), relevance: nil, character: .spouty))
                .previewContext(WidgetPreviewContext(family: .accessoryRectangular))
                .previewDisplayName("Rectangular")
            EmojiRangerWidgetEntryView(entry: SimpleEntry(date: Date(), relevance: nil, character: .spouty))
                .previewContext(WidgetPreviewContext(family: .accessoryInline))
                .previewDisplayName("Inline")

#if os(iOS)
```

Implement the new intent recommendation API.  Whiel itents are fully configurable in editing UI on iOS, on watchOS, we need to provide a preconfiguredl ist.  Override new `recommendations` method.
```swift
return recommendedIntents()
            .map { intent in
                return IntentRecommendation(intent: intent, description: intent.hero!.displayString)
            }
```

New widget families are smaller than iOS widgets.  Consider teh content of your applications.

# Making glanceable views
Let's go to the view.  We can see code for systemSmall, let's add code for accessoryCircular.
```swift
ProgressView(interval: entry.character.injuryDate...entry.character.fullHealthDate,
                         countdown: false,
                         label: { Text(entry.character.name) },
                         currentValueLabel: {
                Avatar(character: entry.character, includeBackground: false)
            })
            .progressViewStyle(.circular)
```

System will keep oru ProgresView updated, avoiding the need for multiple timeline entries.

```swift
case .accessoryRectangular:
        HStack(alignment: .center, spacing: 0) {
            VStack(alignment: .leading) {
                Text(entry.character.name)
                    
                Text("Level \(entry.character.level)")
                Text(entry.character.fullHealthDate, style: .timer)
            }.frame(maxWidth: .infinity, alignment: .leading)
            Avatar(character: entry.character, includeBackground: false)
        }
```

Important that you use the default font parameters and make use of font styles.  Different between iOS and watchOS.  Will sit on screen adjacent to tohers, we recommend using title, headline, body, and caption.

headline => first line
body => additional detial text
caption => basically this is the "small text headline" like "MON" 6
title => basically this is the "large text headline" like MON "6"

**Inline accessories are drawn according to system-defined coloring and font.**

Supply multiple views from lengthy to small.  Chooses first view that fits.
```swift
ViewThatFits {
                Text("\(entry.character.name) is resting, combat-ready in \(entry.character.fullHealthDate, style: .relative)")
                Text("\(entry.character.name) ready in \(entry.character.fullHealthDate, style: .timer)")
                Text("\(entry.character.avatar) \(entry.character.fullHealthDate, style: .timer)")
            }
```

Refer to [[Compose custom layouts with SwiftUI]] for more.

# Privacy
We've discussed active state of your widgets and complications.  Youl'l need to consider whether the device is redacting content or in a low-luminance state.

|               | Sensitive data visible       | content redacted   |
| ------------- | ---------------------------- | ------------------ |
| awake         | unredacted and awake         | refacted and awake |
| low luminance | unredacted and low luminance | redacted and low luminance                   |

Users can opt into redacted states in settings.  Ensure your content is prepared for both readacted and low luminance.  Ensure your complications/widgets work in all cases.

```swift
@Environment(\.isLuminanceReduced)
var isLuminanceReduced

var body: some View {
    if isLuminanceReduced {
        Text("🙈").font(.title)
    } else {
        Text("🐵").font(.title)
    }
}
```

You can now prepare always-on content for every timeline entry, not just one.

Low update cadence of always on.  Use environment value to remove any time-sensitive content and optimize content for lower frequency.

Redaction.  By default, privacy mode will show a redacted version.  If you have some elements thata re sensitive, you can use the `privacySensitive` modifier to mark only some views.  
```swift
VStack(spacing: -2) {
    Image(systemName: "heart")
        .font(.caption.bold())
        .widgetAccentable()
    Text("\(currentHeartRate)")
        .font(.title)
        .privacySensitive()
}
```


# Wrap up
* widgets have been inspired by complications
* Bring that inspriation to iOS

[[Compose custom layouts with SwiftUI]]

* https://developer.apple.com/documentation/widgetkit/adding_widgets_to_the_lock_screen_and_watch_faces
* https://developer.apple.com/documentation/WidgetKit