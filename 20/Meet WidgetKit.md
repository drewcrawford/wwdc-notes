#widgetkit
# What makes a great widget?
* Glanceable
* Relevant
* Personalized

## Glanceable
Our widgets come in many sizes.  Especially at the smallest size, you only have 4x homescreen icons.

People only spend a few moments on their homescreen.  They shouldn't need to interact with your widget.

**Widgets are not mini-apps**.  Think of this about *content*.

[[Design great widgets]]

## Relevant
Stacks use on-device intelligence
Siri shortcuts donation
WidgetKit API

[[Add configuration and intelligence to your widgets]]

## Personalization
* 3 sizes: s,m,l.  You're not required to support all the sizes, but I recommend supporting as many as possible.
* Tapping while in edit mode, flips around to get more configuration.
* Uses intents

We can generate the entire configuration UI from your intent.

# Topic
*The average person goes to the homescreen over 90x a day*.

Dont' show loading screens.  WidgetKit extensions are background extensions that return a series of view hierarchies in a timeline.  This avoids the load stuff.

# How WidgetKit works
* Timeliine allows views to be ready up front
* Can refresh from your main app
* Schedule updates from extension

# Defining a widget
## `kind`
A single stocks extension, provides an experience of stock overview.
But also that same extension provides a detail widget.


## `configuration`
Static configuration -> Doesn't need any configuration
Intent configuration -> widget settings

## `supportedFamilies`
`systemSmall`, medium, or large
 ## `placeholder`
 Each kind of widget is required to provide
 Default content of the widget.  
 No user data
 Queried on environment change.  e.x, dyanmic type
 
 
 Basically this is the launch screen of the widget.
 
  ## example

 
 ```swift
 @main
 public struct SampleWidget: Widget {
 	private let kind: String = "SampleWidget"
	public var body: some WidgetConfiguration {
		StaticConfiguration(kind: kin,
		provider: Provider(),
		placeholder; PlaceholderView()) {entry in
			SampleWidgetEntryView(entry: entry)
		}
		.configurationDisplayName("My Widget")
		.description("This is an example widget.")
	}
 }
 ```
 
# Creating a glanceable experience
StatelessUI
Widgets are not mini-apps
No scrolling
No videos or animated images
Tap interactions are ok -> deep link into the app

The entire widget can be associated with a URL link using the windget URL.
Can create sublinks in system medium or large with `Link`

# Views, timelines, and reloads

## Placeholder (discussed earlier)
## Snapshot
Quickly display a single entry.  Used for gallery.

## Timeline
many snapshots

Return for both dark and light.  We will take the view and serialize to disk.  We JIT render each entry.  

Typically return a day's worth of content.  However for fresher data we can reload.

System will wake up exetension and ask for a new timeline.

## Code

```swift
public protocol TimelineProvider {
	associatedtype Entry: TimelineEntry
	typealias Context = TimelineProviderContext
	func snapshot(with context: Self.Context, completion: @escaping(Self.Entry) -> ())
	func timeline(with context: Self.Context, completion: @escaping(Timeline<Self.Entry>) -> ())
}
```

## `ReloadPolicy`
* `atEnd` (of the timeline)
* `after(date: Date)`
* `never` -> maybe you need user data before getting a timeline

Widgets viewed frequently will receive more reloads
Environment changes.  e.g. timezone

## App-driven reloads
* background notification
* user changes data in the app

Can use `reloadTimelines(ofKind:)`

Only reload your data when needed.

Background reloads are budgeted.  Be efficient.

# Personalization & intelligence

* Intents framework
* Parameters
* Used with Siri, shortcuts, and now widgets

New in iOS 14, your app can answer intents questions, not just extensions.

[[What's new in sirikit and shortcuts]]

## Code
Use `Intentsconfiguration`
conform to `IntentTimelineProvider`.

* Shortcuts donation

[[Add configuration and intelligence to your widgets]]
`TimelineEntryRelevance` -> score, duration

Relative to all entries oyu have ever provided.

# Wrap up
Widgets are not mini-apps
Glanceable
Timelines, reloads, and intelligence

