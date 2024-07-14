Learn the principles of the App Intents framework, like intents, entities, and queries, and how you can harness them to expose your app's most important functionality right where people need it most. Find out how to build deep integration between your app and the many system features built on top of App Intents, including Siri, controls and widgets, Apple Pencil, Shortcuts, the Action button, and more. Get tips on how to build your App Intents integrations efficiently to create the best experiences in every surface while still sharing code and core functionality.

# Friction vs flow
Reduce friction wherever you can.  Ex, switching apps.

Each is in their own box.  Only way to see/do anything in a box is to go into it.  Leave your current box and go into another one.  Switch apps.

A whole family of system features making your experience easier to flow.
My example app.  Catalog of trails in the area.  Open to specific place in app.

Widget?  

# Understanding the framework

Common foundation for building features.  Siri, spotlight, shortcuts.  They take the core features from inside your app and present them outside your app.  To do that, you need to expose your app's core features in a way the system understands.  Two big parts.  

First, define what your core actions and content are.
Communciation with your app.

Intents -> verb
Entities -> nouns
app shortcut -> sentence.  


# building the code

* shortcuts action
* parameterized action
* HS widget
* control center control
* spotlight/siri


Two required pieces:
* localizable title
* perform method
Intents should always be meaningful actions.  
Always have a result, but it may be empty.

`openAppWhenRun` .  

## parameterized query

Parameters ought to be entity.  

Entity has 3 things
* display representation
* ID
* query - turns actions asking for entities into actual entities.

1.  What entities are there?
2. what entity has this ID?

`EnumerableEntityQuery` -> `allEntities`.  Simplest form of query.  App intents can derive the more complicated ones from this, if you build for iOS 18 sdk.  Your whole model has to fit into memory!

## widget

Conditions can/do change routinely.  handy to keep an eye on them.  Use a widget.  Glanceable information.  ex just shows pinned trail, but let's keep an eye on a few different trails, to pick which one is nicest.

Configurable widget -> trail parameter, much like open trail action.

[[Explore enhancements to App Intents]]

`WidgetConfigurationIntent`.  

## controls

[[Extend your app’s controls across the system]]

`ControlWidget` protocol.  `some ControlWidgetConfiguration`.

`AppIntentControlConfiguration`.  

## spotlight and siri

App shortcut.  Wrapper that you, the developer, create around an intent that highlights it as an important function of your app.

Offered by spotlight, action button, apple pencil pro, etc.

`AppShortcutsProvider`.  List of app shortcuts.  Here we wrap the intent we created earlier.  Instance, not a type.  Pre-fill some or all parameters.

phrases -> must contain app name
shortTiel, systemImageName.

No registration code.  App intents framework automatically detects the provider and handles registration.

Use `result` to return data so we don't open the app.

conform return type to `ProvidesDialog & ShowsSnippetView`.  

# Wrap up
* make your app flow
* Adopt features like Siri, Shortcuts, widgets, and more
* All based on app intents

[[Design App Intents for system experiences]]
[[Bring your app to Siri]]
[[What’s new in App Intents]]



# Resources
* https://developer.apple.com/documentation/AppIntents/AcceleratingAppInteractionsWithAppIntents
* https://developer.apple.com/documentation/AppIntents
* https://developer.apple.com/documentation/AppIntents/Creating-your-first-app-intent
* https://developer.apple.com/forums/topics/machine-learning-and-ai?cid=vf-a-0010
* https://developer.apple.com/documentation/AppIntents/Making-actions-and-content-discoverable-and-widely-available
