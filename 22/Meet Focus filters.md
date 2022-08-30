Select either a system or custom focus.

Notification behavior can be customized.  During work focus, someone only allows notifications from coworkers.  Or select few appst hat are relevant to work.

For each focus, behavior can be configured and scheduled in settings.

# Focus filters
Customize app behavior based on currently-enabled focus.  

Examples.  Calendar allows peopel to filter which calendars should be visible by default when a focus is enabled.  Provides a visual indication that we have focused content.

Mail notifications are also filtered.  I can set up mail to only deliver work-related mail notifications during the work focus and prevent personal mail notifications from interrupting.

## Use cases
* multiple accounts
* Filtering content
* Preventing distraction
* Appearance

## how focus filters work
Your app defines what can be customized by a user per focus, with app intent.
System exposes what can be configured.  UI to configure properties will be exposed in focus settings as a focus filter.
Users can configure focus filters for your app.  In focus settings.
# Defining a focus filter
Implement `SetFocusFilterIntent`.  Indicates your app is interested in having custom settings per focus.
Define your parameters
Set display representation

```swift
// Implementing SetFocusFilterIntent

import AppIntents

struct ExampleChatAppFocusFilter: SetFocusFilterIntent {

    static var title: LocalizedStringResource = "Set account, status & look"
    static var description: LocalizedStringResource? = """
        Select an account, set your status, and configure
        the look of Example Chat App.
    """
}
```

Before your filter is configured, it will be surfaced to the user.  Icon is your app icon.  Primary text is your app's name.  Secondary text matches title variable in your filter.

System includes description string you provided for additional context.  Static, read by system at the time your app is installed.

## parameters
* Provide a series of proeprties decorated as parameters
* name and datatype
* Custom data types defined as entities
[[Dive into App Intents]]

Only specify data types and names for each parameter.  Up to users to configure the value of the parameter that would apply during each focus

* parameters can be marked optional
	* provide default values if not optional

```swift
// Defining your Parameters & Entities

import AppIntents

struct ExampleChatAppFocusFilter: SetFocusFilterIntent {

    @Parameter(title: "Use Dark Mode", default: false)
    var alwaysUseDarkMode: Bool

    @Parameter(title: "Status Message")
    var status: String?

    @Parameter(title: "Selected Account")
    var account: AccountEntity?

    // ...
}
```

In focus settings, once a user configures the focus filter, it's presented similarly to what I showed earlier.  Here we have dynamic content in the pills, reflecting what was configured.

Primary text: Represent what parameters have been configured
Secondary text: Represent what those values are we configured them to.

```swift
// Display Representation

struct ExampleChatAppFocusFilter: SetFocusFilterIntent {
    // ...
  
    var localizedDarkModeString: String {
        return self.alwaysUseDarkMode ? "Dark" : "Dynamic"
    }

    var displayRepresentation: DisplayRepresentation {
        var titleList: [LocalizedStringResource] = [], subtitleList: [String] = []
        if let account = self.account {
            titleList.append("Account")
            subtitleList.append(account.displayName)
        }
        if let status = self.status {
            titleList.append("Status")
            subtitleList.append(status)
        }
        titleList.append("Look")
        subtitleList.append(self.localizedDarkModeString)
    
        let title = LocalizedStringResource("Set \(titleList, format: .list(type: .and))")
        let subtitle = LocalizedStringResource("\(subtitleList.formatted())")

        return DisplayRepresentation(title: title, subtitle: subtitle)
    }
  
    // ...
}
```
# Acting on a focus filter
How can your app know what someone has customized?  Update itself accordingly?

When system has determined its' important for your app to know about this change, it will deliver either by
1.  If you're running, `.perform`.
2. If app is not running, you can implement an extension to get spun up.

Since perform can get called either way, not every app needs an extension. If your app only updates its own view in response to a focus transition, just the app is ok.

* widgets
* notifications
* badges

generally consider extensions for these.  Or anything outside your app's own views.

* implement perform
* access populate parameter values
* update your app

Your implementation of `perform` is called when we determine you need to respond.  Also called when system determines that previously delivered values are no longer relevant.  In this case your parameters are configured with a default value.

```swift
// Implementing Perform on your Focus filter

import AppIntents

struct ExampleChatAppFocusFilter: SetFocusFilterIntent {
    // ...

    func perform() async throws -> some IntentResult {
        let myData = AppData(
            alwaysUseDarkMode: self.alwaysUseDarkMode,
            status: self.status,
            account: self.account
        )
        myModel.shared.updateAppWithData(myData)
        return .result()
    }
  
    // ...
}
```

You may need to query the current focus filter parameters.
```swift
// Calling Current

import AppIntents

func updateCurrentFilter() async throws {
    do {
        let currentFilter = try await ExampleChatAppFocusFilter.current
        let myData = AppData(
            myRequiredBoolValue: currentFilter.myRequiredBoolValue,
            myOptionalStringValue: currentFilter.myOptionalStringValue,
            myOptionalAppEnum: currentFilter.myOptionalAppEnum,
            myAppEntity: currentFilter.myAppEntity
        )
        myModel.shared.updateAppWithData(myData)
    } catch let error {
        print("Error loading current filter: \(error.localizedDescription)")
        throw error
    }
}
```

# Providing additional context
You can influence your app behavior outside your app's views.
* notifications filtering
* setting badge count
Give the system context.
* return it when `perform` is called.
* Set it anytime
## notification filtering
Set `filterPredicate` in the `AppContext`.  Pass `filterCriteria` on the `UNNotificationContent`.  If notification does not match filter predicate, it is silenced.

```swift
// Set filterPredicate on an App context

import AppIntents

struct ExampleChatAppFocusFilter: SetFocusFilterIntent {

    var appContext: FocusFilterAppContext {
        let allowedAccountList = [account.identifier]
        let predicate = NSPredicate(format: "SELF IN %@", allowedAccountList)
        return FocusFilterAppContext(notificationFilterPredicate: predicate)
    }
}
```
then configure the notification:
```swift
// Pass filterCriteria on UNNotificationContent

let content = UNMutableNotificationContent()
content.title = "Curt Rothert"
content.subtitle = "Slide Feedback"
content.body = "The run through today was great. I had few comments about slide 22 and 28."
content.filterCriteria = "work-account-identifier"
```

I guess the gist of this design is we don't let apps know what your active focus is necessarily.  They only get their properties.

## Setting badge count
* prevents distraction
* Call `setBadgeCount` on the `UNUserNotificationCenter`

# Next steps
* Implement a focus filter
* Determine configurable properties
* Act on focus filter
* Provide additional context



* https://developer.apple.com/forums/tags/wwdc2022-10121
* https://developer.apple.com/forums/create/question?&tag1=362030&tag2=388030
* https://developer.apple.com/documentation/AppIntents/focus
* https://developer.apple.com/documentation/usernotifications

