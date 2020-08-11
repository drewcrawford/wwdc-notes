#swiftui 
#widgetkit 

# SwiftUI in widgets
Provide a timeline of content for the home screen to display
Create widgets for macOS and iOS with a single implementation
Great opportunity to use and learn SwiftUI

App to log and track caffeine intake and give me an estimate of how much caffeine is in my body.  *Wired*.

* app's visual identity and color scheme
* Amount of caffeine in my body
* Last drink I had, and how long ago it was.
* Shape of the background is concentric with the widget
* Duration at the bottom to update live


# Demo
`WidgetPreviewContext` to preview the widget in xcode previews.
Using `Color` as a child in `ZStack` can create a background color for the widgets.

Can use `Text("\(whatever,style:.relative)")` to get updating text on the widget.

Rounded rectangles.  Could use `.cornerRadius()` and try to find the value that looks good.  But different devices may use a different radius.  Better way is the conatiner-relative shape.

`.background(ContainerRelativeShape().fill(Color...))`  This will take on the appropriate radius in a system-defined way.  As I change padding, the corner radius will change to stay in sync visually with the widget.

For a great widget experience, provide a placeholder that can be used when the device is locked etc.  `.isPlaceholder(true)`.  SwiftUI knows to replace the content of text with rounded rect.    Specific text that's just static text can be `.isPlaceholder(false)`.

Widgets come in different size families.  Add `@Environment(\.widgetFamily) var widgetFamily`.



# New APIs

## corner radius
Usually you don't want the same corner radius, rather you want a radius that is concentric.
```swift
// Concentric corner radius with ContainerRelativeShape

struct PillView : View {
    var title: Text
    var color: Color

    var body: some View {
        Text(title)
            .background(ContainerRelativeShape().fill(color))
    }
}
```

## displaying date and time

```swift
// Displaying date and time

// June 3, 2019
Text(event.startDate, style: .date)

// 11:23PM
Text(event.startDate, style: .time)

// 9:30AM - 3:30PM
Text(event.startDate...event.endDate)

// +2 hours
// -3 months
Text(event.startDate, style: .offset)

// 2 hours, 23 minutes – Automatically updating as time pass
Text(event.startDate, style: .relative)

// 36:59:01 – Automatically updating as time pass
Text(event.startDate, style: .timer)
```

These are automatically updated as time passes, which are a great way to make your widgets feel alive on the home screen.

* compelling widget experiences
* swiftUI support for adaptive layouts
* new API for dates, shapes, and links.  Also work in apps!

[[Meed WidgetKit]]
[[Widgets Code-Along Pt 1]]





