#localization

# Translation
Millions of users open weather every day.
UI is also adapted to left-to-right, etc.

Declare the string using string localized.
```swift
let windPerceptionLabelText = String(
    localized: "Wind is making it feel cooler", 
    comment: "Explains the wind is lowering the apparent temperature"
)
```

If i open the context menu of an email, i can move it to archive.  Works for ipad in english, but other languages have different words for the verb vs the noun.

When they appear in different contexts,
```swift
let filter = String(localized: "Archive.label",
                 defaultValue: "Archive", 
                      comment: "Name of the Archive folder in the sidebar")

let filter = String(localized: "Archive.menuItem",
                 defaultValue: "Archive", 
                      comment: "Menu item title for moving the email into the Archive folder")
```
(new this year)

[[Streamline your localized strings]]

Soemtimes the same english word or even an entire sentence is shown in different contexts in the UI.  Make sure to use different strings.

```swift
String(localized: "Show weather in \(locationName)", 
       comment: "Title for a user activity to show weather at a specific city")

String(localized: "Show weather in My Location",
       comment: "Title for a user activity to show weather at the user's current location")
```

Use string interpolation to insert any location name.  In other languages we might run itno grammatical issues.  e.g., it works for a city name but not for the current location.

Use two different strings.  This ensures that translators are able to use the correct grammar for their language.  Inserting a variable can have impact on the entire sentence.
* capitalization
* inflect the grammar
* etc

## Comments

```swift
String(localized: "Show weather in \(locationName)",
         comment: "Title for a user activity to show weather at a specific city")
```
A comment is really important for translators.  make sure to give them the context they need to translate it, keeping the same intention as you had.  
* which element it's shown in, label, button, etc.
* what context (where it is shown), section header, context menu, user activity
* What each variable is

Translators might not see the app at rutnime when translating, but you should be able to create a shared understanding through comments.

## Remote content
When content is downloaded, it should be presented in the language the user prefers.  What you can do to ensure users are able to see remote content.

1.  Can send a list of supported languages to the app.
2. Device knows which language the user prefers.
3. Leverage apple's framework with 

```swift
let allServerLanguages = ["bg", "de", "en", "es", "kk", "uk"]
let language = Bundle.preferredLocalizations(from: allServerLanguages).first
```

Can request remote content in a given language.

Rain or shine, the weather app is rich in data.  many aspects contain numbers and counts.

```swift
String(localized: "\(amountOfRain) in last \(numberOfHours) hour",
         comment: "Label showing how much rain has fallen in the last number of hours")

String(localized: "\(amountOfRain) in last ^[\(numberOfHours) hour](inflect: true)",
         comment: "Label showing how much rain has fallen in the last number of hours")
```

In english we need to use plural form if larger than one.  In ukranian, even more variants.
Leverage apple's frameworks.

Declare the string in code, use a stringsdict file which encodes.
[[Streamline your localized strings]]

**Is there no count in the sentence?  Don't use a plural rule.**  Ex:
```swift
if selectedCount == 1 {
    return String(localized: "Remove this city 
                              from your favorites")
} else {
    return String(localized: "Remove these cities 
                              from your favorites")
}
```

If the string **does** include a number, consider using a plural rule.

```swift
String(localized: "\(amountOfRain) in last ^[\(numberOfHours) hour](inflect: true).",
         comment: "Label showing how much rain has fallen in the last number of hours")
```

# Formatters
Display the current humidity in percent.

```swift
let humidity = 54

// In a SwiftUI view
Text(humidity, format: .percent)

// In Swift code
humidity.formatted(.percent)
```

That's all!  Formatter takes care of everything else.  % sign, numbering system, etc.

Only the beginning of what type of data you can format.  Formatters for everything.

```swift
date.formatted(
    .dateTime.year()
    .month()   
) // Jun 2022

whatToExpect.formatted()
// New features, exciting API, and advanced tips

amountOfRain.formatted(
    .measurement(
        width: .narrow,
        usage: .rainfall)) // 12mm

(date...<later).formatted(
    .components(
        style: .wide
    )
) // 24 minutes, 18 Seconds

date.formatted(
    .relative(
        presentation: 
            .numeric
    )
) // 2 minutes ago

let components = PersonNameComponents()
…
nameComponentsFormatter
    .string(from: components)
// Andreas Neusüß or 田中陽子

excitementLevel.formatted(
    .number
    .precision(
        .fractionLength(2)
    )
) // 1,001.42

price.formatted(
    .currency(
        code: "EUR"
    )
) // $20.99

distance.formatted(
    .measurement(
        width: .wide,
        usage: .road)
) // 500 feet

bytesCopied.formatted(
    .byteCount(
        style: .file
)) // 42.23 MB
```

[[Formatters Make Data Human-Friendly]]


## Numbers in a string

In spanish, we vary "50mm in next 24h" based on singular or plural.

String 2mm: formatter.
embed result: sentence.
```swift
func expectedPrecipitationIn24Hours(for valueInMillimeters: Measurement<UnitLength>) -> String {
    // Use user's preferred measures
    let preferredUnit = UnitLength(forLocale: .current, usage: .rainfall)

    let valueInPreferredSystem = valueInMillimeters.converted(to: preferredUnit)

    // Format the amount of rainfall
    let formattedValue = valueInPreferredSystem
        .formatted(.measurement(width: .narrow, usage: .asProvided))

    let integerValue = Int(valueInPreferredSystem.value.rounded())

    // Load and use formatting string
    return String(localized: "EXPECTED_RAINFALL", 
               defaultValue: "\(integerValue) \(formattedValue) expected in next \(24)h.", 
                    comment: "Label - How much precipitation (2nd formatted value, in mm or Inches) is expected in the next 24 hours (3rd, always 24).")
}
```

Interesting use of `\(24)` here.  Key is declared in a stringsdict.
```xml
Localizable.stringsdict English:

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>EXPECTED_RAINFALL</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%#@next_expected_precipitation_amount_24h@</string>
        <key>next_expected_precipitation_amount_24h</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>d</string>
            <key>other</key> <! We don't need to define a plural in English>
            <string>%2$@ expected in next %3$dh.</string>
        </dict>
    </dict>
</dict>
</plist>
```
Localizable.stringsdict Spanish:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>EXPECTED_RAINFALL</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%#@next_expected_precipitation_amount_24h@</string>
        <key>next_expected_precipitation_amount_24h</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>d</string>
            <key>one</key>
            <string>Se prevé %2$@ en las próximas %3$d h.</string>
            <key>other</key>
            <string>Se prevén %2$@ en las próximas %3$d h.</string>
        </dict>
    </dict>
</dict>
</plist>
```


# Swift packages

Sometimes your strings are in a dependency.  Or maybe you distribute your own code with packages.

```swift
let package = Package(
    name: "FoodTruckKit",

    defaultLocalization: "en",

    products: [
       .library(
            name: "FoodTruckKit",
            targets: ["FoodTruckKit"]),
    ],
    …
)
```

Can declare default localization.
Xcode now reads that and recognizes that you're interested in providing a localized experience.  Will add options to export localizations to product menu.  Now it also works for swift packages.

`.xcloc` files.  To import, use improt localization.  Xcode will place the files at the correct file paths. Workflow is now identical to localizing your app.

Loading a string requries that you specify bundle `.module`
```swift
let title = String(localized: "Wind",
                      bundle: .module, 
                     comment: "Title for section that
                               shows data about wind.")
```

[[Swift packages Resources and Localization - 20]]

## Library author
* Localize your swift package
* Advertise supported languages

App developer
* Ensure that your dependencies are localized
* test your app

# Layout and SwiftUI
English vs arabic.  Not only translations are adapted to the languaeg but also the layout.

[[Get it right ... to left]]

You'll notice the script in hindi is taller.  System does this automatically, make sure you odn't give elements a fixed height.  Please expect your text to change according to circumstances.

10-day forecast.
* position of elements according to longest label
* Double the size in some cases
* Do not have fixed spacing to their neighbor elements
* Keep in mind that labels need to be flexible.

Also expect labels to grow horizontally with a longer translation.  It can be a challenge to accomomdate.  This year, SwiftUI adds support for `Grid` to help you build this element more easily.
```swift
// Requires data types "Row" and "row" to be defined

struct WeatherTestView: View {
    var rows: [Row]
    var body: some View {
        Grid(alignment: .leading) {
            ForEach(rows) { row in
                GridRow {
                    Text(row.dayOfWeek)
                    
                    Image(systemName: row.weatherCondition)
                        .symbolRenderingMode(.multicolor)
                    
                    Text(row.minimumTemperature)
                        .gridColumnAlignment(.trailing)
                    
                    Capsule().fill(Color.orange).frame(height: 4)
                    
                    Text(row.maximumTemperature)
                }
                .foregroundColor(.white)
            }
        }
    }
}
```
When the label needs more space, we move all labels.  SwiftUI does all the heavy lifting.

How to make view with longer translation work with a limited amount of space.  To fix this, we do not remove the icon.  The solution is to use 2+ lines of text if needed, which is the default behavior.  we do not encourage you to change that.

Mail app accomomdates in a creative way.  In sheet presentation there are 4 buttons.  When a translation of one title is too long, we move to 2x2 grid.

This year, we add `ViewThatFits`.  Provide an alternative layouts if space is too constrained.

```swift
ViewThatFits {
	firstChoice
	secondChoice
}
```

Hiding a view because translation is too long is bad practice.  First, try having a flexible layout.

Also for smaller/larger text.

[[Compose custom layouts with SwiftUI]]

Adapting layout can be a challenge but it gets easier this year.

# Wrap up
* It's not always like English
* Format your values
* Localize your swift package
* make your layout flexible



* https://developer.apple.com/documentation/Xcode/localizing-package-resources
* https://developer.apple.com/documentation/Xcode/localization
* https://developer.apple.com/localization/
* https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/Introduction/Introduction.html