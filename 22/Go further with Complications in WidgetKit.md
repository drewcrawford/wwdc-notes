#widgetkit 

[[Complications and Widgets Reloaded]]
[[Build complications in SwiftUI]]

# Unique to watchOS
In iOS 16, we brought complication-style widgets to the phone's lock screen.  In watchOS 9, we brought widgetkit.

* watch specific family
* auxiliary content
* multiple representations

Multiple ways of being rendered dependingo n face.  Sometimes flat, sometimes curved, etc.

In addition to 3 new complication styles, accessoryRectangular, accessoryCircular, and accessoryInline, we have an acccessoryCorner.  Can either be shown as a large ciruclar content.  Or a smaller circular content with a curved label or gauge.  

New view modifier.
```swift
struct CornerView: View {
    let value: Double

    var body: some View {

        ZStack {
            AccessoryWidgetBackground()
            Image(systemName: "cup.and.saucer.fill")
                .font(.title.bold())
                .widgetAccentable()
        }

    }
}
```

We use `.widgetLabel`.

```swift
struct CornerView: View {
    let value: Double

    var body: some View {

        ZStack {
            AccessoryWidgetBackground()
            Image(systemName: "cup.and.saucer.fill")
                .font(.title.bold())
                .widgetAccentable()
        }
        .widgetLabel {
            Gauge(value: value,
              in: 0...500) {
                Text("MG")
            } currentValueLabel: {
                Text("\(Int(value))")
            } minimumValueLabel: {
                Text("0")
            } maximumValueLabel: {
                Text("500")
            }
        }


    }
}
```

for accessoryCorner can cspecify a gague, progress view, or text in your widget's label?

On accessoryCircular family.  On infographic watch face.  How to add text n the dial, in addition to the ciruclar view we have here.

More appropriate to move my gauge to the center so it's not redundant with the information in the circle area.

```swift
struct CircularView: View {
    let value: Double

    var body: some View {

        Gauge(value: value,
              in: 0...500) {
            Text("MG")
        } currentValueLabel: {
            Text("\(Int(value))")
        }
        .gaugeStyle(.circular)

    }
}
```

```swift
struct CircularView: View {
    let value: Double

    var body: some View {
        let mg = value.inMG()

        Gauge(value: value,
              in: 0...500) {
            Text("MG")
        } currentValueLabel: {
            Text("\(Int(value))")
        }
        .gaugeStyle(.circular)
        .widgetLabel {
            Text("\(mg, formatter: mgFormatter) Caffeine")
        }

    }

    var mgFormatter: Formatter {
        let formatter = MeasurementFormatter()
        formatter.unitOptions = [.providedUnit]
        return formatter
    }
}

extension Double {
    func inMG() -> Measurement<UnitMass> {
        Measurement<UnitMass>(value: self, unit: .milligrams)
    }
}
```

Let's switch circular content to the SFSymbol

```swift
struct CircularView: View {
    let value: Double

    var body: some View {
        let mg = value.inMG()

        ZStack {
            AccessoryWidgetBackground()
            Image(systemName: "cup.and.saucer.fill")
                .font(.title.bold())
                .widgetAccentable()
        }
        .widgetLabel {
            Text("\(mg, formatter: mgFormatter) Caffeine")
        }

    }

    var mgFormatter: Formatter {
        let formatter = MeasurementFormatter()
        formatter.unitOptions = [.providedUnit]
        return formatter
    }
}

extension Double {
    func inMG() -> Measurement<UnitMass> {
        Measurement<UnitMass>(value: self, unit: .milligrams)
    }
}
```

But now when I switch to a face that doesn't have a bezel with info, I've lost all the info.  So that's not ideal.  We can toggle with `showsWidgetLabel`.

```swift
struct CircularView: View {
    let value: Double
    @Environment(\.showsWidgetLabel) var showsWidgetLabel

    var body: some View {
        let mg = value.inMG()
        if showsWidgetLabel {
            ZStack {
                AccessoryWidgetBackground()
                Image(systemName: "cup.and.saucer.fill")
                    .font(.title.bold())
                    .widgetAccentable()
            }
            .widgetLabel {
                Text("\(mg, formatter: mgFormatter) Caffeine")
            }
        }
        else {
            Gauge(value: value,
                  in: 0...500) {
                Text("MG")
            } currentValueLabel: {
                Text("\(Int(value))")
            }
            .gaugeStyle(.circular)
        }

    }

    var mgFormatter: Formatter {
        let formatter = MeasurementFormatter()
        formatter.unitOptions = [.providedUnit]
        return formatter
    }
}

extension Double {
    func inMG() -> Measurement<UnitMass> {
        Measurement<UnitMass>(value: self, unit: .milligrams)
    }
}
```

Now I can have the papropriate level of information.

There's one more way to be aware of.  The XL watch face.  Now it supports a single large, circular complication.  `.accessoryCircular` scales to match the style of the face.  Note, as this face has a single large complication, do not use the increased canvas style to densely-pack your complication.

two widget families more
* accessoryrectangular => none of these show the widget label
* accesoryinline => already as a widget label.  Watch face extract iamges/text and renders them itself to match look of the face


# Migration

* adopt widgetkit
* upgrade existing instaleld complications

framework differences
* widgetkit instead of clockkit
* different set of families

| ClockKit                                       | WidgetKit            |
| ---------------------------------------------- | -------------------- |
| CLKComplicationFamilyGraphicRectangular        | accessoryRectangular |
| CLKComplicationFamilyGraphicCorner             | accessoryCorner      |
| GraphicCircular,GraphicBezel,GraphicExtraLarge | accessoryCircular    |
| UtilitarianSmallFlat, UtilitarianLarge         | accessoryInline                     |

* views instead of templates
* timelines

## WidgetKit
[[Meet WidgetKit]]

## Migration API
System asks for migrations every time a new face is shared?

CLKComplicationDataSource.widgetMigrator.  Be that yuor complication datasource itself, or some other type.


```swift
var widgetMigrator: CLKComplicationWidgetMigrator {
    self
}
```

this is its method.  If it provides a static migration, use the static migration configuration.
```swift
func widgetConfiguration(from complicationDescriptor: CLKComplicationDescriptor) async -> CLKComplicationWidgetMigrationConfiguration? {
    CLKComplicationStaticWidgetMigrationConfiguration(kind: "CoffeeTracker", extensionBundleIdentifier: widgetBundle)
}
```
This other one for intents?

```swift
func widgetConfiguration(from complicationDescriptor: CLKComplicationDescriptor) async -> CLKComplicationWidgetMigrationConfiguration? {
    CLKComplicationIntentWidgetMigrationConfiguration(kind: "CoffeeTracker", extensionBundleIdentifier: widgetBundle, intent: intent, localizedDisplayName: "Coffee Tracker")
}
```

You will also need to include intent definitions in watch **app**.

# Wrap up
* new ways to develop complications
* simplify experience


https://developer.apple.com/documentation/WidgetKit/Converting-A-ClockKit-App
https://developer.apple.com/documentation/WidgetKit/Creating-lock-screen-widgets-and-watch-complications
https://developer.apple.com/documentation/widgetkit/adding_widgets_to_the_lock_screen_and_watch_faces
https://developer.apple.com/forums/tags/wwdc2022-10051
https://developer.apple.com/forums/create/question?&tag1=286&tag2=330&tag3=494030
https://developer.apple.com/documentation/WidgetKit

