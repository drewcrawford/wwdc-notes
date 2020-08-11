We find that people all over the world are using their device in languages that may not be traditionally associated in that region.

More language and region combinations.

Formatter APIs -> can save you tons of time and effort.

* dates and times
* measurements
* names
* lists
* numbers
* strings

# Dates and times
```swift
// Dates and Times

// Date with Day/Month/Year and Time
let dateFormatter = DateFormatter()
dateFormatter.dateStyle = .medium
dateFormatter.timeStyle = .short
dateFormatter.string(from: Date())

// Day of Week + Date + Month
let dateFormatter = DateFormatter()
dateFormatter.setLocalizedDateFormatFromTemplate
    ("MMMMdEEEE")
dateFormatter.string(from: Date())

// Abbreviated Day of Week
let dateFormatter = DateFormatter()
dateFormatter.setLocalizedDateFormatFromTemplate
    ("ccccc")
dateFormatter.string(from: Date())
```

How do templates work?

https://www.unicode.org/reports/tr35/tr35-dates.html#Date_Field_Symbol_Table

Not reproducing the template table.

Order of the fields in the (localized) template does not matter.  
Formatter's job to assemble into something that makes sense for the locale.

You are supposed to do this

```swift
dateformatter.setLocalizedDateFormatFromTemplate("dMMM")
dateFormatter.dateFormat = DAteFormatter.dateFormat(fromTemplate: "dMMM")
```

and not this

```swift
dateFormatter.dateFormat = "dMMM"
```

I should note here, most of my projects want to specify an exact (english) date format, not necessarily thinking about the internationalization situation, but they perceive any variance in the english locale as a bug.  That is the real reason these APIs are used, and we still seem to lack a solution for it.

## date components
```swift
// Dates and Times

// Date and Time Components
let formatter = DateComponentsFormatter()
formatter.unitsStyle = .abbreviated
let components = DateComponents(hour: 2, minute: 26)
formatter.string(from: components)

// Date and Time Intervals
let formatter = DateIntervalFormatter()
formatter.dateTemplate = "dMMM"
formatter.string(from: startDate, to: endDate)

// Relative Dates and Times
let formatter = RelativeDateTimeFormatter()
formatter.dateTimeStyle = .named
formatter.localizedString(from: DateComponents(day: -1))
```

# Measurements
```swift
// Measurements

// Temperature
let formatter = MeasurementFormatter()
let temperature = Measurement<UnitTemperature>
    (value: 16, unit: .celsius)
formatter.numberFormatter.maximumFractionDigits = 0
formatter.string(from: temperature)

// Speed
let speed = Measurement<UnitSpeed>
    (value: 14, unit: .kilometersPerHour)
formatter.string(from: speed)

// Pressure
let pressure = Measurement<UnitPressure>
    (value: 1.01885, unit: .bars)
formatter.string(from: pressure)
```

[[Measurements and Units - 16]]

# Names
```swift
// Names

let formatter = PersonNameComponentsFormatter()
var nameComponents = PersonNameComponents()
nameComponents.familyName = "Iwasaki"
nameComponents.givenName = "Akiya"
nameComponents.nickname = "Aki-chan"

// Full Name
formatter.string(from: nameComponents)

// Short Name: Respects User Preferences
formatter.style = .short
formatter.string(from: nameComponents)

// Abbreviated Name - for monograms for example
formatter.style = .abbreviated
formatter.string(from: nameComponents)
```

## monograms
```swift
// Abbreviated Name: Monogram
formatter.style = .abbreviated
let monogram = formatter.string(from: nameComponents)
if (monogram.count <= 2) {
    // Use Monogram
}
else {
    // Use Icon
}
```

But they cannot be generated for all names.  Can restrict based on length check `.count` works ok.
Of course, a character count cannot indicate how tall/wide the string would be, so we may need a graphical check.

## name formatter
```swift
/ Names

let formatter = PersonNameComponentsFormatter()
var nameComponents = PersonNameComponents()
nameComponents.familyName = "岩崎"
nameComponents.givenName = "晃也"
nameComponents.nickname = "あきちゃん"

// Full Name
formatter.string(from: nameComponents)

// Short Name: Respects User Preferences
formatter.style = .short
formatter.string(from: nameComponents)

// Abbreviated Name
formatter.style = .abbreviated
formatter.string(from: nameComponents)
```

Will handle this correctly even if name is different from iPhone language.

# Lists

```swift
// Lists

// English Localization

let items = [ "English", "French", "Spanish" ] ListFormatter.localizedString(byJoining: items)

let items = [ "English", "Spanish" ] ListFormatter.localizedString(byJoining: items)

let items = [ "Spanish", "English" ] ListFormatter.localizedString(byJoining: items)

// Spanish Localization

let items = [ "Inglés", "Español" ] ListFormatter.localizedString(byJoining: items)

let items = [ "Español", "Inglés" ] ListFormatter.localizedString(byJoining: items)
```

ingles y espanol vs espanol e ingles

# Numbers
```swift
// Numbers

let formatter = NumberFormatter()
formatter.numberStyle = .decimal
formatter.string(from: 32.768) // French (France)

let formatter = NumberFormatter()
formatter.numberStyle = .decimal
formatter.string(from: 32.768) // Arabic (Egypt)

formatter.percentSymbol

formatter.decimalSeparator
```

## number formatter
```swift
// Numbers

let formatter = NumberFormatter()
formatter.numberStyle = .percent
formatter.string(from: 0.71) // English (US)

let formatter = NumberFormatter()
formatter.numberStyle = .percent
formatter.string(from: 0.71) // Turkish (Turkey)
```

## strings
```swift
// Strings

var body: some View {
    Text("\(photosCount) Photos Selected")
}
```

then create a stringsdict.

https://help.apple.com/xcode/mac/current/#/devd9af5f7ae


Takeaway: if you correctly localize strings using stringsdict, your app will shine regardless of whether you support only one language or 100.

Sample app that you can download and play with.

# Resources
https://developer.apple.com/localization/
https://help.apple.com/xcode/mac/current/#/devd9af5f7ae
https://www.unicode.org/reports/tr35/tr35-dates.html#Date_Field_Symbol_Table
https://developer.apple.com/documentation/foundation/formatter/displaying_human-friendly_content

