Follow along as we build a widget for the Smart Stack on watchOS 10 using the latest SwiftUI and WidgetKit APIs. Learn tips, techniques, and best practices for creating widgets that show relevant information on Apple Watch.

code-along preparation.  
download sample code
Backyard Birds.xcodeproj

Widgets
* backyardVisitorsWidget.swift
* AppIntent.swift

# Widget configuration
widget structure is where widget configuration is defined.  new in watchOS, our app intent configurations.  

configuration intent provider and view were all stubbed out when we created the widget extension.  We'll look at each one and implement for our widget.  This widget definition looks good, so let's move on to our widget configuration intent.

our widget is using an app intent configuration, to allow it to do 2 things.  First, our widget can provide a set of preconfigured widget gallery.  In the case of backyard birds, we'll provide a configuration for each yard in our app.  Second, the widget configuration intent is used to specify dates when our widget is most relevant.  prioritized in smart stack.

I've already added `backyardID` parameter.  One widget intent for each yard.

[[Explore enhancements to App Intents]]
[[Dive into App Intents]]
# Timeline setup
will hold all the data our widget views will need to render themselves for a particular date.

Locate the generated simple entry structure.  The date and configuration proeprties were added when this file was generated.  We need to define any interaction properties our widget views will need.  

name, food, water status.  If a bird is visiting, it wills how the visting bird and its name.



### 4:15 - TimelineEntry
```swift
struct SimpleEntry: TimelineEntry {
    var date: Date
    var configuration: ConfigurationAppIntent
    var backyard: Backyard
    
    var bird: Bird? {
        return backyard.visitorEventForDate(date: date)?.bird
    }
    
    var waterDuration: Duration {
        return Duration.seconds(abs(self.date.distance(to: self.backyard.waterRefillDate)))
    }

    var foodDuration: Duration {
        return Duration.seconds(abs(self.date.distance(to: self.backyard.foodRefillDate)))
    }

    var relevance: TimelineEntryRelevance? {
        if let visitor = backyard.visitorEventForDate(date: date) {
            return TimelineEntryRelevance(score: 10, duration: visitor.endDate.timeIntervalSince(date))
        }
        return TimelineEntryRelevance(score: 0)
    }
}
```

relevance
* score - prioritize an entry against other entries in the same timeline.  we set to 10, to rank an entry with a visitor higher than an entry without a visitor.  Arbitrary.
* duration - tell the smart stack how long the entry is valid.  This lasts until the visitor's end date.
### 7:50 - placeholder function
```swift
func placeholder(in context: Context) -> SimpleEntry {
  return SimpleEntry(date: Date(), configuration: ConfigurationAppIntent(), backyard: Backyard.anyBackyard(modelContext: modelContext))
}
```

used when the widget displays for the first time.  Should return quickly.  since we updated our timeline entry, we need to supply one.  Can provide a random backyard from the app's data model.

### 8:15 - snapshot function
```swift
func snapshot(for configuration: ConfigurationAppIntent, in context: Context) async -> SimpleEntry {
  if let backyard = Backyard.backyardForID(modelContext: modelContext, backyardID: configuration.backyardID) {
    if let event = backyard.visitorEvents.first {
      return SimpleEntry(date: event.startDate, configuration: configuration, backyard: backyard)
    } else {
      return SimpleEntry(date: Date(), configuration: configuration, backyard: backyard)
    }
  }

  let yard = Backyard.anyBackyard(modelContext: modelContext)
  return SimpleEntry(date: Date(), configuration: ConfigurationAppIntent(), backyard: yard)
}
```

used when a widget in transient situations.  Using sample data is fine.



new in xcode, we can preview a widget timeline.  here we see a preview of the rectangular widget, and a series of timeline entries, etc.
# Widget views
using the default view that was generated when we added the widget.  Lets build out our view to visualize our timelines.

### 10:26 - Widget Entry View
```swift
struct BackyardBirdsWidgetEntryView: View {
    @Environment(\.widgetFamily) private var family
    var entry: SimpleEntry
    
    var body: some View {
        switch family {
        case .accessoryRectangular:
            RectangularBackyardView(entry: entry)
        default:
            Text(entry.date, style: .time)
        }
    }
}
```


### 11:23 - Backyard Rectangular View
```swift
struct RectangularBackyardView: View {
    var entry: SimpleEntry
    
    var body: some View {
        HStack {
            if let bird = entry.bird {
                ComposedBird(bird: bird)
                    .scaledToFit()
                    .widgetAccentable()
                    .frame(width: 50, height: 50)
                VStack(alignment: .leading) {
                    Text(bird.speciesName)
                        .font(.headline)
                        .foregroundStyle(bird.colors.wing.color)
                        .widgetAccentable()
                        .minimumScaleFactor(0.75)
                    Text(entry.backyard.name)
                        .minimumScaleFactor(0.75)
                    HStack {
                        Image(systemName: "drop.fill")
                        Text(entry.waterDuration, format: remainingHoursFormatter)
                        Image(systemName: "fork.knife")
                        Text(entry.foodDuration, format: remainingHoursFormatter)
                    }
                    .imageScale(.small)
                    .minimumScaleFactor(0.75)
                    .foregroundStyle(.secondary)
                }
                .frame(maxWidth: .infinity, alignment: .leading)
            } else {
                Image(.fountainFill)
                    .foregroundStyle(entry.backyard.backgroundColor)
                    .imageScale(.large)
                    .scaledToFit()
                    .widgetAccentable()
                    .frame(width: 50, height: 50)
                VStack(alignment: .leading) {
                    Text(entry.backyard.name)
                        .font(.headline)
                        .foregroundStyle(entry.backyard.backgroundColor)
                        .widgetAccentable()
                        .minimumScaleFactor(0.75)
                    HStack {
                        Image(systemName: "drop.fill")
                        Text(entry.waterDuration, format: remainingHoursFormatter)
                        Image(systemName: "fork.knife")
                        Text(entry.foodDuration, format: remainingHoursFormatter)
                    }
                    .imageScale(.small)
                    .minimumScaleFactor(0.75)
                    Text("\(entry.backyard.historicalEvents.count) visitors")
                        .minimumScaleFactor(0.75)
                        .foregroundStyle(.secondary)
                }
                .frame(maxWidth: .infinity, alignment: .leading)
            }
        }
        .containerBackground(entry.backyard.backgroundColor.gradient, for: .widget)
    }
}
```

this will be the widget shown in the smart stack.  We'll ahve an image on the left, and 3 lines of text on the right.

Uses a timeline entry we modified earlier.  Before we continue, let's switch our canvas view to the smart stack rectangular view.

Let's update the composed bird.  Make the view scale to fit, and make it widget accentable so it will tint on a watch face that is tinted.

fonts, widgetAccentable, etc.  minimumScaleFactor.  Set the foreground style of the last line to secondary.

apply these same updates to the view in the else statement when there isn't a bird, etc.

New in swiftui is the container background.  Let's replace with a gradient.  This is selectively used by the system, here will only appear in the watchos smart stack, not on the watch face.  

# Timeline
Let's finish building out the timeline.  

### 16:30 - Timeline Function
```swift
func timeline(for configuration: ConfigurationAppIntent, in context: Context) async -> Timeline<SimpleEntry> {
  var entries: [SimpleEntry] = []

  if let backyard = Backyard.backyardForID(modelContext: modelContext, backyardID: configuration.backyardID) {
    for event in backyard.visitorEvents {
      let entry = SimpleEntry(date: event.startDate, configuration: configuration, backyard: backyard)
      entries.append(entry)
      let afterEntry = SimpleEntry(date: event.endDate, configuration: configuration, backyard: backyard)
      entries.append(afterEntry)
    }
  }
  return Timeline(entries: entries, policy: .atEnd)
}
```

at the top of the function is an array of timeline entries.  Use this to build our timeline.  First, let's remove the generated timeline code.

Let's get the configured yard using backyard ID app intent.  Backyard structure has a property that contians all the visitor events of that yard.

note we need to create an entry for when visitors leave too, because those should also be on the timeline.

here we need to implement an array of intent recommendations.  backyard ID, etc.

Remove the default implementation. L et's create an array of recommendations to return.  

### 18:35 - Recommendations Function
```swift
func recommendations() -> [AppIntentRecommendation<ConfigurationAppIntent>] {
  var recs = [AppIntentRecommendation<ConfigurationAppIntent>]()

  for backyard in Backyard.allBackyards(modelContext: modelContext) {
    let configIntent = ConfigurationAppIntent()
    configIntent.backyardID = backyard.id.uuidString
    let gardenRecommendation = AppIntentRecommendation(intent: configIntent, description: backyard.name)
    recs.append(gardenRecommendation)
  }

  return recs
}
```

now provides a list of widget configurations.  Whent he person is selecting the birds widget, etc.  
# Relevance

You've now built a widget on watchOS taht will surface as a watch-based complication.

earlier, we talked about relevance, but there's more we can do.  Each yard keeps track of the water and food available for birds.  Our new widget also shows that information.  We can provide the system a list of relevant intents during the time period when we know water or food is running low.  Our widget will be prioritzed during those times.


### 20:47 - Relevant Intents Function
```swift
func updateBackyardRelevantIntents() async {
    let modelContext = ModelContext(DataGeneration.container)
    var relevantIntents = [RelevantIntent]()
    
    for backyard in Backyard.allBackyards(modelContext: modelContext) {

        let configIntent = ConfigurationAppIntent()
        configIntent.backyardID = backyard.id.uuidString
        let relevantFoodDateContext = RelevantContext.date(from: backyard.lowSuppliesDate(for: .food), to: backyard.expectedEmptyDate(for: .food))
        let relevantFoodIntent = RelevantIntent(configIntent, widgetKind: "BackyardVisitorsWidget", relevance: relevantFoodDateContext)
        relevantIntents.append(relevantFoodIntent)

        let relevantWaterDateContext = RelevantContext.date(from: backyard.lowSuppliesDate(for: .water), to: backyard.expectedEmptyDate(for: .water))
        let relevantWaterIntent = RelevantIntent(configIntent, widgetKind: "BackyardVisitorsWidget", relevance: relevantWaterDateContext)
        relevantIntents.append(relevantWaterIntent)
    }

    do {
        try await RelevantIntentManager.shared.updateRelevantIntents(relevantIntents)
    } catch { }
}
```

we'll create a relevant context based on dates.  In this case, we'll use the backyards' future low and empty food dates.  we create a relevant intent, an dappend to our array.

now we do the same thing for low/empty water dates.  now the relevant intent manager has date ranges when each possible configuration has higher relevance.



### 23:00 - Update Relevant Intents
```swift
Task {
  await updateBackyardRelevantIntents()
  WidgetCenter.shared.reloadTimelines(ofKind: "BackyardVisitorsWidget")
}
```

we've now built a widget for smart stack.  and updated relevant intents, to priorize our widget when it's most relevant.

[[Design widgets for the Smart Stack on Apple Watch]]
[[Explore enhancements to App Intents]]
[[Complications and Widgets Reloaded]]


* https://developer.apple.com/design/human-interface-guidelines/widgets/overview/introduction/
* https://developer.apple.com/documentation/WidgetKit
* https://developer.apple.com/documentation/watchOS-Apps/updating-your-app-and-widgets-for-watchos-10
