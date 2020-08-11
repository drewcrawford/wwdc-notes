#widgetkit 
# Families
* small
* medium
* large

`supportedFamilies`
Can use `.widgetFamily` environment value to figure out which family you're in.


# Timelines
Timeline provider is the engine of our widget.

```swift
struct TimelineReloadPolicy {
atEnd 
after(time)
never //system will not update the widget, we have to reload
}
```

There may be many timelines active at a time, each with its own reload policy.

The system intelligently schedules updates
# Configuration
Driven-by SiriKit.  
`INIntent`.  Specifically, custom intents.

[[Add configuration and intelligence to your widgets]]

## Intent definition
Category: View
Intent is eligible for widgets

Widget type needs to be an `IntentConfiguration`.
Need `IntentTimelineProvider` conformance rather than `TimelineProvider`.

# Deep linking
`systemSmall` is a single tap area, whereas medium and large can define custom areas with `Link`.

`.widgetURL`

# Updating our widget

[[Design great widgets]]
[[build swiftui views for widgets]]
[[Widgets Code-along Pt 3]]

