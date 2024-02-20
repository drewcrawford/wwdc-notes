Learn how to make animated and interactive widgets for your apps and games. We'll show you how to tweak animations for entry transitions and add interactivity using SwiftUI Button and Toggle so that you can create powerful moments right from the Home Screen and Lock Screen.

interactivity - directly manipulate widget
animation - bring widgets to life by helping user get a sense of how content has changed as a result of their action.

# Animations
system will animate with adefault animation.  

how animation works with widgets.

In a regular swiftui app, you use state to drive changes to your view.  Animation are driven by state mutation, using modifier like `withAnimation`.  But widgets work slightly differently.  Widgets don't have state, instead they create a timeline made of entities which correspond to different view rendered at a specific time.

swiftui diffs and animates the parts that have chagned.  By default, widget gets an implicit spring animation and content transitions.  But you can use all transitions, animations, content transition provided by swiftui.

[[Explore SwiftUI animation]]


#widgetkit  #swiftui 

### Usage for the container background modifier - 3:54
```swift
.containerBackground(for: .widget) {
    Color.cosmicLatte
}
```

normally to see a widget animationg, you need a bunch of entries and wait for them to appear onscreen.  That can be tedious.  We havea  great solution, the preview API.  

### Define a preview for the caffeine tracker widget - 4:22
```swift
#Preview(as: WidgetFamily.systemSmall) {
    CaffeineTrackerWidget()
} timeline: {
    CaffeineLogEntry.log1
    CaffeineLogEntry.log2
    CaffeineLogEntry.log3
    CaffeineLogEntry.log4
}
```



check out the session [[Build programmatic UI with Xcode Previews]] to learn more.


### Add a numeric text content transition - 5:41
```swift
struct TotalCaffeineView: View {
    let totalCaffeine: Measurement<UnitMass>

    var body: some View {
        VStack(alignment: .leading) {
            Text("Total Caffeine")
                .font(.caption)

            Text(totalCaffeine.formatted())
                .font(.title)
                .minimumScaleFactor(0.8)
                .contentTransition(.numericText(value: totalCaffeine.value))
        }
        .foregroundColor(.espresso)
        .bold()
        .frame(maxWidth: .infinity, alignment: .leading)
    }
}
```

### Set up transition on LastDrinkView - 6:21
```swift
struct LastDrinkView: View {
    let log: CaffeineLog

    var body: some View {
        VStack(alignment: .leading) {
            Text(log.drink.name)
                .bold()
            Text("\(log.date, format: Self.dateFormatStyle) · \(caffeineAmount)")
        }
        .font(.caption)
        .id(log)
        .transition(.push(from: .bottom))
    }

    var caffeineAmount: String {
        log.drink.caffeine.formatted()
    }

    static var dateFormatStyle = Date.FormatStyle(
        date: .omitted, time: .shortened)
}
```


id - whenever the log changes, it's a new view.


### Configuring animation for the transition - 7:18
```swift
struct LastDrinkView: View {
    let log: CaffeineLog

    var body: some View {
        VStack(alignment: .leading) {
            Text(log.drink.name)
                .bold()
            Text("\(log.date, format: Self.dateFormatStyle) · \(caffeineAmount)")
        }
        .font(.caption)
        .id(log)
        .transition(.push(from: .bottom))
        .animation(.smooth(duration: 1.8), value: log)
    }

    var caffeineAmount: String {
        log.drink.caffeine.formatted()
    }

    static var dateFormatStyle = Date.FormatStyle(
        date: .omitted, time: .shortened)
}
```


# Interactivity
execute action right from widget.  Before we jump into xcode, let's discuss architecture.

Create a better mental model for how interactivity works.  When you create a widget extension... discovered by the system.  Run as an independent process.

widgets define a timeline provider that returns a series of entries which are effectively the widget model.  If a widget is visible, system launches thee xtension process and asks the timeline provider for entries.

fed back into the viewbuilder.  Used to generate a series of views based on these entries.  Then the system saves these and archives to disk.  When tiem to display a specific entry, system decodes and renders the widget representation in the system process.

**your view code only runs during archiving**.  But if your data is not static, you might want to update those entries.

Do that by callign `reloadTimelines` function.  Whenever updating data displayed by your widget.  regenerate new entries, new copies, etc.
### Reload the timeline for a widget - 9:18
```swift
WidgetCenter.shared.reloadTimelines(ofKind: "LocationForecast")
```

key takeaways

* widgets are rendered in a separate process
* changes are driven by timeline entries
* reloads from interactions are guaranteed

### App intent to log a caffeine drink - 13:06
```swift
import AppIntents

struct LogDrinkIntent: AppIntent {
    static var title: LocalizedStringResource = "Log a drink"
    static var description = IntentDescription("Log a drink and its caffeine amount.")

    @Parameter(title: "Drink", optionsProvider: DrinksOptionsProvider())
    var drink: Drink

    init() {}

    init(drink: Drink) {
        self.drink = drink
    }

    func perform() async throws -> some IntentResult {
        await DrinksLogStore.shared.log(drink: drink)
        return .result()
    }
}
```

since widgets are rendered in a different process, swiftui won't execute your closure in your process space.  We need a way to represent actions that can be executed by the widget extension.  Thankfully, there's a solution already, app intents.

same intent can be used to represent the action in a widget.  at its core, app intents are a protocol letting you define in code action sthat can be executed by the system.  here, we define an app intent to toggle a todo item.

intent defines a number of parameters as inputs.  async function called `perform`, where you will have the busines slogic to run your intent.

intents are very powerful, much mroe to know about them.

[[Dive into App Intents]]
[[Explore enhancements to App Intents]]

when you improt both swiftui and appintents, there's a new family of initializers on `Button` and `Toggle` that accept an app intent as an argument.  

note that only these controls are supported, others won't work.
these initializers also work in regular apps, so you can share logic between your widget and your app.

note that `perform` is an async function.  Takea dvantage if you're doing any asynchronous work.  as soon as `perform` returns, system initiates load of timeline.  Giving you the opportunity to update content of your widget.  Persist all information necessary to load your updated widget before returning from `perform`.

improtant to use the property wrapper beacuse only stored properties are persisted.  

important ecosystem benefit of using app intents.  This intent I've just defined, is also available in shortcut and siri.  So the investment in defining it here will pay dividends beyond widgets.

New view holding our buttons.  In this view... 
### Create view to log a new drink - 15:10
```swift
struct LogDrinkView: View {
    var body: some View {
        Button(intent: LogDrinkIntent(drink: .espresso)) {
            Label("Espresso", systemImage: "plus")
                .font(.caption)
        }
        .tint(.espresso)
    }
}
```

ltitle tip here, you can actually directly build target for widget extension.  xcode will install the widget right on the homescreen for you.  my widget now has the button I've just defined.  demo.

### Use the invalidatable content modifier - 16:28
```swift
struct TotalCaffeineView: View {
    let totalCaffeine: Measurement<UnitMass>

    var body: some View {
        VStack(alignment: .leading) {
            Text("Total Caffeine")
                .font(.caption)

            Text(totalCaffeine.formatted())
                .font(.title)
                .minimumScaleFactor(0.8)
                .contentTransition(.numericText(value: totalCaffeine.value))
                .invalidatableContent()
        }
        .foregroundColor(.espresso)
        .bold()
        .frame(maxWidth: .infinity, alignment: .leading)
    }
}
```

when we reload timeline, it can cause latency.  even more pronounced with widget on mac.  In ym widget, the value shown is the amount of caffeine won't update until updated entry arrives.  Can fix with `invalidatableContent`.  This uses a system effect to show the value is out of date until an update arrives.  This improves the perception of latency

* use for values that are invalidated by the interaction
* Use `Toggle` to optimistically update a boolean state
	* we prerender the toggle style in both configurations
* If you define your own toggle style, check `configuration.isOn` and use that to switch appearance.

Infuse new life into your widgets.  With widgets now... bring little, delightful interactions to users wherever they are

* fine tune your widget animations
* surface the most important actions
[[Bring widgets to new places]]
