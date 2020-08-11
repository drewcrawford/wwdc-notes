**Complications are as important as the app experience on the watch.**

# Timelines
A representation of your complication's data over time
Enables ClockKit to query your app once and get all information needed
Extend or invalidate as necessary to ask ClockKit to requery your app

* Each tour is associated with one time, rather than a range.
* Times are earlier than the tour's start time.  So the user has a heads up.

Not all data will behave like this.  A temperature forecast wants to have temps at each forecasted time.

# Complication building blocks
Graphic Extra Large (new)
Graphic families allow for more visually-oriented types of data.

Ideally you want to support as many complication families as you can.

Templates represent different layouts within families.  Despire being associated with a specific family, each family inherits from `CLKCom;licationTemplate`.

You can find out more in the documentation.

`CLKComplicationTimelineEntry` -> date, `complicationTemplate`.

Your main interaction is by `CLKComplicationDataSource`.



# Providing data
Just return the given entry:

```swift
// CLKComplicationDataSource - Required
class ComplicationController: NSObject, CLKComplicationDataSource {

    func getCurrentTimelineEntry(
        for complication: CLKComplication, 
        withHandler handler: @escaping (CLKComplicationTimelineEntry?) -> Void)     
    {
        // Call the handler with the current timeline entry
        handler(createTimelineEntry(forComplication: complication, date: Date()))
    }
}
```

Provide timeline entry in the future:

```swift
// CLKComplicationDataSource - Timeline Support
extension ComplicationController {

	//specifies how far in the future you can provide entries
    func getTimelineEndDate(
        for complication: CLKComplication, 
        withHandler handler: @escaping (Date?) -> Void) 
    {
        handler(timeline(for: complication)?.endDate)
    }
	
	//provide as many entries as is appropriate, up to the limit, after the date.
	//date is the last timeline entry that we already have
	//limit 
    func getTimelineEntries(
        for complication: CLKComplication, 
        after date: Date, 
        limit: Int, 
        withHandler handler: @escaping ([CLKComplicationTimelineEntry]?) -> Void) 
    {
       handler(timeline(for: complication)?.entries(after: date, limit: limit))
    }
}
```

## reloading

If all your entries are invalid, you can call `reloadTiemline(for complication:)`.

If your previous entires are still valid, and want to let us know you can provide more, call `extendTimeline(for complication:)`.

## Providing data
* complication data needs to adapt to different sizes, layouts, and styles
* space is often constrained
* tell us your intentions
* we handle the formatting

## text providers
`CLKDateTextProvider` -> avoid truncating date text.  If space is constrained, we fall back to shorter versions.

```swift
let longDate: Date = DateComponents(year: 2020, month: 9, day: 23).date ?? Date()
let units: NSCalendar.Unit = [.weekday, .month, .day]
let textProvider = CLKDateTextProvider(date: longDate, units: units)
```

for range questions, we use `CLKRelativeDateTextProvider`.  Will auto-update text for whatever the current time is.

```swift
let timerStart: Date = …
let units: NSCalendar.Unit = [.hour, .minute, .second]
let textProvider = CLKRelativeDateTextProvider(date: timerStart, style: .timer, units: units)
```

`CLKTimeTextProvider` -> shows time
`CLKTimeIntervalTextProvider` -> Shows range of time
`CLKSimpleTextProvider` -> displays any text

## image providers
Provides images for multiple contexts
One-piece for single color watch faces
Two-piece for multicolor watch faces, composed of a background and foreground.

The grahpic families ask for `CLKFullColorImageProvider`.  But in some contexts, the graphic contexts are tinted.  By default, we desaturate, but you can override.

[[Explring Tinted Graphic Complications - 19]]

## gauge providers
customize for colors, gradients, adn fill fraction
`CLKTimeIntervalGaugeProvider` enables an auto-updating gauge based on a start and end date.

## #swiftUI in complications
All templates where you can use `CLKFullColorImageProvider` allow SwiftUI alternatives
Reuse components from your app
Easier to stand out and create unique complications
[[Build complications in SwiftUI]]

# Multiple complications
* One app can provide many complications
* Great way to show multiple data points on the watch face
* Fill a watch face with your complications and share with your users

`CLKComplciationDescriptor`
* identifier
* `displayName` shown during watch face editing
* `supportedFamilies`
* Optional, mutually-exclusive properties: `userInfo` or `userActivity`.



```swift
// CLKComplicationDataSource - Multiple Complication Support
extension ComplicationController {
    var descriptors : [CLKComplicationDescriptor] = []
    var dataDict = Dictionary<AnyHashable, Any>()
        
    for station in data.stations {
        dataDict = [“name": station.name, “shortName": station.shortName]
        descriptors.append(
            CLKComplicationDescriptor(
                identifier: station.name,
                displayName: station.name,
                supportedFamilies: CLKComplicationFamily.allCases,
                userInfo: dataDict))
    }
    
    descriptors.append(
        CLKComplicationDescriptor(
            identifier: "LogSighting",
            displayName: "Log Sighting",
            supportedFamilies: CLKComplicationFamily.allCases))

    descriptors.append(
        CLKComplicationDescriptor(
            identifier: "SeasonData",
            displayName: "Season Data",
            supportedFamilies: [.graphicRectangular]))
        
    // Call the handler with the currently supported complication descriptors
    handler(descriptors)
}
```

If you update this list to indicate that you no longer support a complication, but the user still has it on their watch face, we will continue to call you even though you don't support it.  Do your best to continue providing data for this case.

* Tapping a complication launches your app
* Complication descriptors with a `userActivity` will be launched with it
* Either way, we pass some info in the user dictionary

`CLKLaunchedTimelineEntryDateKey: Date(...),
CLKLaunchedComplicationIdentiferKey: "Makena beach"`
And of course your specified entries.

This all defines what complications you **support**.  To get the actual **data**, you use the appropriate method.

## The default complication identifier
We have something called the "default complication identifier".

If you had a complication before watchOS 7 and a user has it on their watch face
Or the user shares the complication but chooses to remove complication data,

you'll be asked about the identifier `CLKDefaultComplicationIdentifier`.  **Even if you don't explicitly support it in your descriptors.**

You should support this. Show teh same data as your complication before watchOS 7.  Or show the most popular data in your app
Or just show your app icon.  


# Example
This template will be used to select... face editing.  AS well as apple watch app on iPhone.  Use sample data
```swift
func getLocalizableSampleTemplate(
    for complication: CLKComplication, 
    withHandler handler: @escaping (CLKComplicationTemplate?) -> Void) 
{
    let template = createSampleTemplate(forComplication: complication)
    handler(template)
}
```


```swift
func createTimelineEntry(
    forComplication complication: CLKComplication, 
    date: Date) -> CLKComplicationTimelineEntry? 
{
    guard let template = createTemplate(forComplication: complication, date: date) else {
        return nil
    }
    return CLKComplicationTimelineEntry(date: date, complicationTemplate: template)
}
```
templates

```swift
func createTemplate(
    forComplication complication: CLKComplication, 
    date: Date) -> CLKComplicationTemplate? 
{
    var station: Station? = nil
    if let stationName = complication.userInfo?["name"] as? String {
        station = data.stations.first(where: { $0.name == stationName })
    }
    
    let image = UIImage(named: "Spout-small")!
    let spoutFullColorImageProvider = CLKFullColorImageProvider(fullColorImage: image)
    let logSightingTextProvider = CLKSimpleTextProvider(
        text: "Log Sighting", 
        shortText: "Log")
	//fall back if something unexpected happens
    let defaultTemplate: (CLKComplicationFamily) -> CLKComplicationTemplate = { family -> CLKComplicationTemplate in
      // Return a default complication template for the given family
    }
  
  	
    switch (complication.family, complication.identifier) {
	
	//create a graphic rectangular full template
    case (.graphicRectangular, "SeasonData"):
        return CLKComplicationTemplateGraphicRectangularFullView(
            ChartView(
                seriesData: data.last7DaysSightings, 
                seriesColor: .turquoise)
	
    case (.graphicCircular, "LogSighting"):
        return CLKComplicationTemplateGraphicCircularStackImage(
            line1ImageProvider: spoutFullColorImageProvider, 
            line2TextProvider: logSightingTextProvider)
	
    case (.graphicCircular, _):
        guard let station = station else { return defaultTemplate(.graphicCircular) }
        return CLKComplicationTemplateGraphicCircularView(
            SightingTypeView(station: station))
          
    case (.graphicCorner, _):
        guard let station = station else { return defaultTemplate(.graphicCorner) }
        return CLKComplicationTemplateGraphicCornerTextImage(
            textProvider: station.timeAndShortLocTextProvider, 
            imageProvider: station.whaleActivityFullColorProvider)
                
    case (.graphicExtraLarge, _):
        guard let station = station else { return defaultTemplate(.graphicExtraLarge) }
        return CLKComplicationTemplateGraphicExtraLargeCircularStackText(
            line1TextProvider: station.timeAndLocationTextProvider, 
            line2TextProvider: station.shortLocationTextProvider)

	//if we are called about a template taht we don't know, such as the default
	//return the default template
    default:
        return defaultTemplate(complication.family)
    }
}
```

# wrap up
* Make complications
* Provide your data in a timeline
* Customize your content with SwiftUI
* Support the default complication identifier

[[Keep your complications up to date]]
[[Meet watch face sharing]]
[[Developing complications for Apple Watch Series 4 - TT]]

