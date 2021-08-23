#widgetkit 

# Widgets
[[Meet WidgetKit]]
[[Widgets Code-Along Pt 1]]
[[Building SwiftUI views for widgets]]
[[Design Great Widgets]]

# Relevant
* Time
* Presentation
* Location

## Time
Timeline you give us will be archived and rendered in the future.  Lets system have your UX ready.

Because ScreenTime can't predict what will happen, it just uses a single entry.  Simplistic timeline.

But consider multiple entries.  If you have forward-looking content or can forecast your content, take advantage.

Weather's timeline provides multiple entries.  First entry is most accurate, subsequent weather is forecasted data.  Very useful as widget reloads aren't guaranteed.

Photos timeline provides personal/relevant photos at specific times.

Even if you don't have forecastable data, can still incorporate photos that's relevant to the user and surprise/delight.  

### Widget reloads

Individual background reload budgets
Budget updates throughout the day
Influenced by viewing habits
Variable update cadence per widget

A frequently-viewed widgets may receive 40-70 bg updates per day.  roughly every 15-30 minutes if spaced evenly during awake hours.  But our goal was to support varied patterns.

Updates are withheld until budget availability

Reloads are on the order of *minutes and hours*, not seconds.

* TimelineReloadPolicy API (budgeted)
	* Scheduled background updates (atEnd, afterDate:, never)
	* Informs system when you'd like to refresh your widget.
* WidgetCenter reload API
	* Event-based trigger
	* `.reloadTimelines(ofKind:)`, `reloadAllTimelines`. 
	* Normally budgeted, but some exceptions
		* Foreground
		* User session like navigation or now playing audio
		* Supplemental to TimelineReloadPolicy
	* Significant location changes (free)
	* System updates (free)
		* User changes accessibility preference, language/region change, etc.
	
	### Reload policies
	* atEnd
	* afterDate:
	* never

#### `atEnd`
Eligible to be refreshed at end of current timeline.  

Simply the time becomes *eligible* to refreshing, doesn't guarantee exactly at this time.

If using atEnd with single entry timelines, the system will choose an appropriate time.

Recommended for *timelines with sliding windows* on your content.

ex
* Reminders
* calendar
* photos
* tips

Not recommended for *single-entry timelines* as system will choose reload window that may not be what you want
Not recommended for *data that loses accuracy over time* (e.g. weather)

#### `afterDate:`
Reload policy makes your widget eligible for reloading after date is specified.

In full control of eligibility time, we specify when we lose accuracy

* Recommended for unpredictable data
* Recommended for data whose accuracy changes regularly

* stocks
* weather
* news
* mail
* etc

Have to be careful of a few issues
* Be cautious of very frequent reloads
* Watch for unexpected server load with time-alignment
* Consider adding jitter to the date
* Use caching servers

#### `never`
* Recommended for content that only changes through user interaction
* Recommended for content that doesn't change
* Recommended for content gated on conditional access
* Use WidgetCenter API to refresh

TV, notes, music, podcasts, contacts, and more.  Require user interaction to drive content changes, or receive pushes for content updates.

* Leverage timeline entries to your advantage
* Automatic reloads policy
* Event-based relodas via WidgetCenter

## Presentation

Widget may be rerendered iwthout timeline updates.  Great widgets will adapt.

* Color scheme (dark vs light)
* Partial privacy redactions
* Full privacy redactions

### Color scheme

WK will automatically shift between light/dark mode.  Power of SwiftUI.

Not all widgets have to conform to light/dark mode.  e.g. Music and Stocks.  Feel free to continue whatever color scheme makes sense.

```swift
struct MyWidgetEntryView : View {
    var date: Date

    var body: some View {
        ZStack {
            Rectangle().fill(BackgroundStyle())
            VStack {
                Text("Hello")
            }
        }
    }
}

struct MyWidget_Previews: PreviewProvider {
    static var previews: some View {
        MyWidgetEntryView(date: Date())
            .previewContext(WidgetPreviewContext(family: .systemSmall))
            .environment(\.colorScheme, .dark)
    }
}
```

### Partial privacy redactions
* Individual views may now be masked in privacy-sensitive environments

```swift
struct MyWidgetEntryView : View {

    var body: some View {
        ZStack {
            Rectangle().fill(BackgroundStyle())
            VStack(alignment: .leading) {
                Text("Balance")
                    .font(.largeTitle)
                    .fontWeight(.bold)
                    .foregroundColor(Color.blue)
                Text("$128.45")
                    .privacySensitive()
                    .font(.title2)
                    .foregroundColor(Color.gray)
            }
        }
    }
}
```

This modifier can be applied to any view, including container views like HStack and VStack.  

* Store your timeline with 'complete' data protections
* Completely replaces timeline content with placeholder content
* Entitlement
	* `com.apple.developer.default-data-protection: NSFileProtectionComplete`
	
## Location

* Your widget content can be location-aware
* Current location or pre-selected with Intents

### Current location
`Info.plist` requires `NSWidgetUsersLocation` = true
`CLLocationManager` to resolve current location
Use the lowest resolution possible
`isAuthorizedForWidgetUpdates` to check permissions

### permissions
|                              | Widget location accessible?                                            |
|------------------------------|------------------------------------------------------------------------|
| Never                        |                                                                        |
| Ask next time ("allow once") |                                                                        |
| While using app              | Yes (while using)                                                      |
| While using app or widgets   | Yes, widgets can receive up to 15 minutes after widget was last viewed |
| Always                       | Yes                                                                    |

# Customizable
* Size
* Kind
* Configurable

## Size
Support as many sizes as you can.

Use system-standard padding as they vary between sizes.

XL size for iPad.  Same height as large but wider.

If unspecified, we opt in on SDK build.
## Kind
```swift
struct IndividualSymbolWidget : Widget {
    var body: some WidgetConfiguration {
    …
}
}

struct StocksOverviewWidget : Widget {
    var body: some WidgetConfiguration {
    …
    }
}

@main
struct MyWidgetBundle: WidgetBundle {
    var body: some Widget {
        // Order of these widgets defines the order in the Widget Gallery
        IndividualSymbolWidget()
        StocksOverviewWidget()
    }
}
```

Not possible to retract widgets after install. ?
## Configurable
### Static configuration
* Simple
* no configuration
* All instances return the same data

### Intent configurations
* Intents framework
* Parameters
* Also used with Siri and Shortcuts

* System provides configuration UI

### Static ex
```swift
@main
public struct SampleWidget: Widget {
    public var body: some WidgetConfiguration {
        StaticConfiguration(kind: "com.sample.myStaticSampleWidgetKind",
                            provider: Provider()) { entry in
                                SampleWidgetEntryView(entry: entry)
                            }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}

public struct Provider: TimelineProvider {
    public func timeline(with context: Context,
                         completion: @escaping (Timeline<Entry>) -> ()) {
        let entry = SimpleEntry(date: Date())
        // TODO: Generate a timeline entry
        completion(timeline)
    }
}
```

### intent configuration ex
```swift
@main
public struct SampleWidget: Widget {
    public var body: some WidgetConfiguration {
        IntentConfiguration(kind: "com.sample.myIntentSampleWidgetKind",
                            intent: SampleConfigurationIntent.self
                            provider: Provider()) { entry in
                                SampleWidgetEntryView(entry: entry)
                            }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}

public struct Provider: IntentTimelineProvider {
    public func timeline(for configuration: SampleConfigurationIntent, with context: Context,
                         completion: @escaping (Timeline<Entry>) -> ()) {
        let entry = SimpleEntry(date: Date(), configuration: configuration)
        // generate a timeline
        completion(timeline)
    }
}
```

[[Add configuration and intelligence to your widgets]]

# Wrap up
* Keep widget timelines relevant
* Customize through
	* Sizes
	* Kinds
	* Configurations

* https://developer.apple.com/documentation/WidgetKit/Making-a-Configurable-Widget
* https://developer.apple.com/documentation/WidgetKit/Keeping-a-Widget-Up-To-Date

