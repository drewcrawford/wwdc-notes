# Overview
#appintents 

Before now, they had to add to siri.  App shortcuts require no setup.
Available as soon as your app is installed.
Can be run from shortcuts, spotlight, and siri.

# Implement App Shortcuts
Building app shortcuts
* with #appintents 
* swift-only framework
* App intents are implemented in swift
* `AppShortcutsProvider` defines phrases for your Shortcuts
* Must include app name in trigger phrase.
```swift
// StartMeditationIntent creates a meditation session.

import AppIntents

struct StartMeditationIntent: AppIntent {
    static let title: LocalizedStringResource = "Start Meditation Session"

    func perform() async throws -> some IntentResult & ProvidesDialog {
        await MeditationService.startDefaultSession()
        return .result(dialog: "Okay, starting a meditation session.")
    }
}
```

Once the session has started, I return a dialog that is shown to the user.  Localize this string in every locale.

* title specified in sourcecode
* Parameter summary can customize the rendering in shortcuts app, show values inline, etc.

By creating an app shortcut, I can perform the setup step on behalf ot he user, so they can start using the intent as soon as the app is installed.

```swift
// An AppShortcut turns an Intent into a full fledged shortcut
// AppShortcuts are returned from a struct that implements the AppShortcuts
// protocol

import AppIntents

struct MeditationShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: StartMeditationIntent(),
            phrases: ["Start a \(.applicationName)"]
        )
    }
}
```

**Your app can have a maximum of 10 app shortcuts (not intents!)**

Because users may say different phrases, I'll provide some alternative phrases here.  If your app was localized, localize these as well.

```swift
// An AppShortcut turns an Intent into a full fledged shortcut
// AppShortcuts are returned from a struct that implements the AppShortcuts
// protocol

import AppIntents

struct MeditationShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: StartMeditationIntent(),
            phrases: [
                "Start a \(.applicationName)",
                "Begin \(.applicationName)",
                "Meditate with \(.applicationName)",
                "Start a session with \(.applicationName)"
            ]
        )
    }
}
```

If your intent does trigger an app launch, it will **not** be shown in spotlight!!

I want to implement a custom view that siri can show

## custom views
* app intents leverages SwiftUI, similar to widgets.
* Custom app intent views can't include things like interactivity or animations.
* Your app can display custom UI at multiple steps
	* value confirmation
	* intent confirmation
	* after intent finished
* return it from your app intent

```swift
// Custom views give your intent more personality
// and can convey more information

func perform() async throws -> some ProvidesDialog & ShowsSnippetView {
    await MeditationService.startDefaultSession()

    return .result(
        dialog: "Okay, starting a meditation session.",
        view: MeditationSnippetView()
    )
}
```

You'll want your snippet to look like siri UI.

# Add parameters
Ideally, my user can specify session to start when running intent.  I need to extend my intent by adding a parameter to capture a session the user wants to run.
```swift
// An entity is a type that can be used as a parameter
// for an AppIntent.

import AppIntents

struct MeditationSession: AppEntity {
    let id: UUID
    let name: LocalizedStringResource

    static var typeDisplayName: LocalizedStringResource = "Meditation Session"
    var displayRepresentation: AppIntents.DisplayRepresentation {
        DisplayRepresentation(title: name)
    }

    static var defaultQuery = MeditationSessionQuery()
}
```

I need to implement the `AppEntity` protocol.  Tells the app intents framework about my type, lets me specify additional information like how it is displayed.  REquires my type have an identifier.

How to display my entity.  

Need to wire up a default query.

```swift
// Queries allow the App Intents framework to
// look up your entities by their identifier

struct MeditationSessionQuery: EntityQuery {
    func entities(for identifiers: [UUID]) async throws -> [MeditationSession] {
        return identifiers.compactMap { SessionManager.session(for: $0) }
    }
}
```

Must be able to look up entities by id.  So we define this requirement.  We implement the function by looking up sessions.

add a parameter to intent
```swift
// Adding a parameter to an intent allows you to prompt the user
// to provide a value for the parameter

struct StartMeditationIntent: AppIntent {

    @Parameter(title: "Session Type")
    var sessionType: SessionType?

    // ...

}
```

Can specify additional metadata in property wrapper, such as a display name.

Ask the user which session they'd like to run.  Robust support for followup questions.  These prompts willb e displayed anywhere my intent is run.  Siri: speak out the question and ask the user to reply.  In spotlights/shortcuts, user sees touch UI.

* disambiguouation => ask the user to select from a fixed list
* value => ask the user for an open-ended value.  Strings or integers can take any value.
* Confirmation => verify a particular value.  Helpful to double-check that you understand their intent.

But these slow down the conversation.  For more insight into designing great intents, check out [[design App Shortcuts]]

```swift
// Prompting for values can be done by calling methods
// on the property's wrapper type.

func perform() async throws -> some ProvidesDialog {
    let sessionToRun = self.session ?? try await $session.requestDisambiguation(
           among: SessionManager.allSessions,
           dialog: IntentDialog("What session would you like?")
       )
    }
    await MeditationService.start(session: sessionToRun)
    return .result(
       dialog: "Okay, starting a \(sessionToRun.name) meditation session."
    )
}
```

We put the name of the session in the dialog so the user knows we understood their request.

Users prefer siri interactions that are quick and to the point.  Ideally users can tell the session in the initial phrase.  

## Parameterized phrases
* app shortcuts support predefined parameters
* Not meant for open-ended values.  
	* Not possible to gather an arbitrary string from the user in the initial prompt
	* Can't implement "search my diary for X"
	* Specified while your app is running

Components of parameterized prhases
* implement the `suggestedResults()` method on your query
* Notify app intents when your values change
* Create new, parameterized phrases

```swift
// Queries can provide suggested values for your Entity
// that serve as parameters for App Shortcuts

struct MeditationSessionQuery: EntityQuery {
    func entities(for identifiers: [UUID]) async throws -> [MeditationSession] {
        return identifiers.compactMap { SessionManager.session(for: $0) }
    }

    func suggestedEntities() async throws -> [MeditationSession] {
        return SessionManager.allSessions
    }
}
```

Update model layer to notify app intents framework when we change

```swift
// Your app must notify App Intents when your values change
// This is typically best done in your app’s model layer

class SessionModel {
    @Published
    var sessions: [MeditationSession] = []
    private var cancellable: AnyCancellable?

    init() {
        self.cancellable = $sessions.sink { _ in
            MeditationShortcuts.updateAppShortcutParameters()
        }
    }

    // ...

}
```

set phrases
```swift
// Phrases can also contain a single parameter reference

import AppIntents

struct MeditationShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: StartMeditationIntent(),
            phrases: [
                "Start a \(.applicationName)",
                "Begin \(.applicationName)",
                "Meditate with \(.applicationName)",
                "Start a \(\.$session) session with \(.applicationName)",
                "Begin a \(\.$session) session with \(.applicationName)",
                "Meditate on \(\.$session) with \(.applicationName)"
            ]
        )
    }
}
```

these values are pulled from titles.

I'll want to include a few different ways that users might phrase.  smoother experience if the user doesn't remember the phrase.

# Add discoverability

## Natural phrases
* short and memorable
* Consider using your app name as a noun or verb
* App name synonyms can help immensely
	* added in iOS 11.  Now might be a great time.

## Siri Tip
* discoverability is vital for users
* We don't want to lose the discoverabilty benefits of "add to siri" button
* Siri Tip replaces the Add to Siri button.  We recommend replacing this.

Available in #swiftui  and #uikit .  SiriTips are best placed contextually when they're relevant to content onscreen.

Supports dismissal.  Remove the view from your layout and consider not showing again until you feel it's relevant.
`ShortcutsLink` opens the shortcuts app.  

## App isntallation
* app shortcuts appears as soon as your app is installed
	* User may not have logged in
* Parameter values won't be gathered until you notify app intents
	* and launched, etc.
	* If you don't contain any non-parameterized phrases you may not see shortcuts at all
* Siri will show your app shortcuts when asked "What can I do here?" and "What can I do with app?"
* Ordering determined by the order in your source code
	* First phrase in your array is the primary phrase for the app shortcut

# Next steps
* pick great App Shrotcuts for Siri, Spotlight, and the Shortcuts app
* Make your App Shortcuts discoverable

[[design App Shortcuts]]
[[Dive into App Intents]]






* https://developer.apple.com/documentation/AppIntents
