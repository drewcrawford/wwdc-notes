https://developer.apple.com/wwdc23/10027
The widget ecosystem is expanding: Discover how you can use the latest WidgetKit APIs to make your widget look great everywhere. We'll show you how to identify your widget's background, adjust layout dynamically, and prepare colors for vibrant rendering so that your widget can sit seamlessly in any environment.

* desktop - macos
* lock screen
* standby mode
* smart stack - apple watch

your widge can appear here automatically.

People can use your widget on mac even if you don't have a macOS app.

# Transition to content margins
padding applied to widget's body so it doesn't get too close to teh ocntianer.  May vary depending on environment.

watchOS 9 and below, widgets use safe area.  
### SafeAreasWidget - 2:08
```swift
struct SafeAreasWidgetView: View {
    @Environment(\.widgetContentMargins) var margins

    var body: some View {
        ZStack {
            Color.blue
            Group {
                Color.lightBlue
                Text("Hello, world!")
            }
                .padding(margins) 
        }
    }
}

struct SafeAreasWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(...) {_ in
            SafeAreasWidgetView()
        }
        .contentMarginsDisabled()
    }
}
```

on watchOS 10, safe are are now content margins.  This means that modifiers like `ignoresSafeArea` **no longer have any effect in widgets**.

Achieve the same effec twith `contentMarginsDisabled` modifier.

Than add padding back in for widgets that need it.

Can use `widgetContentMargins` environment variable to get the right padding.




# Add a removable background
To make bg color removable, add a containerBackground color.
### EmojiRangerWidget systemSmall - 3:19
```swift
struct EmojiRangerWidgetEntryView: View {
    var entry: Provider.Entry
    
    @Environment(\.widgetFamily) var family

    var body: some View {
        switch family {
        case .systemSmall:
            ZStack {
                AvatarView(entry.hero)
                    .widgetURL(entry.hero.url)
                    .foregroundColor(.white)
            }
            .containerBackground(for: .widget) {
                Color.gameBackground
            }
        }
        // additional cases
    }
}
```

apple watch can also use new container background.
### EmojiRangerWidget accessoryRectangular - 3:48
```swift
var body: some View {
    switch family {
    case .accessoryRectangular:
        HStack(alignment: .center, spacing: 0) {
            VStack(alignment: .leading) {
                Text(entry.hero.name)
                    .font(.headline)
                    .widgetAccentable()
                Text("Level \(entry.hero.level)")
                Text(entry.hero.fullHealthDate, style: .timer)
            }.frame(maxWidth: .infinity, alignment: .leading)
            Avatar(hero: entry.hero, includeBackground: false)
        }
        .containerBackground(for: .widget) {
            Color.gameBackground
        }
    // additional cases
}
```

see [[Design widgets for the Smart Stack on Apple Watch]]

some widgets don't have distinct fg content.  In this case, we can add `containerBackgroundRemovable` to false.

### PhotoWidget - 4:22
```swift
struct PhotoWidget: Widget {
    public var body: some WidgetConfiguration {
        StaticConfiguration(...) { entry in
            PhotoWidgetView(entry: entry)
        }
        .containerBackgroundRemovable(false)
    }
}
```

in standby, we use less padding to make it easier to read far away.



# Dynamically adjust layout
`showsWidgetBackground`.


# Prepare for vibrant rendering


Our system family widgets are shown in vibrant rendering mode on iPad lock screen.  So it's desturated, then colored papropriately for lock screen background.  contrast issues?  

use `widgetRenderingMode` environment variable.

[[Complications and Widgets Reloaded]]

# Wrap up
* the widget ecosystem is expanding!
[[Bring widgets to life]]






