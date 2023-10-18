Discover how to use App Shortcuts to surface frequently used features from your app in Spotlight or through Siri. Find out how to configure search results for your app and learn best practices for creating great App Shortcuts. We'll also show you how to build great visual and voice experiences and extend to other Apple devices like Apple Watch and HomePod. For more information about App Shortcuts and App Intents, check out “Explore enhancements to App Intents” and “Design Shortcuts for Spotlight" from WWDC23.

# Why app shortcuts
* automatically available on app install via
* voice with siri
* spotlight
* shortcuts app

raise the visibility of your app throughout the system

lowers the friction for using your app
helps people discover, remember, and build a habit around your app

"summarize my grocery list with Demo"
# App shortcuts basics

AppIntents framework.  #appintents   Swift-only
define an intent.  Individual task that can be completed with your app, like creating a todo list, etc.

define an app shortcut.  Used with spotlight/siri.  Associate your intent, etc.

Build and test.  

Shortcut to create a new, empty todo list.

1.  Define app intent
2. define app shortcut

code sample not provided.

each app can have at most one `AppShortcutsProvider`.  Intent, phrases, short title, system name.

`@Parameter` to declare variables.

entities/queries.  
Entity -> concepts users want to reference.

In order to find an entityt, we rely on queries.  At runtime, the system instantiates and runs query objects.  Queries return entities.  Entities are used to run intent.

Implement a DisplayRepresentation so the system knows how to describe a specific instance.  Entity needs to have an image or symbol in `DisplayRepresentation`.

Can suggest entities.  System uses suggested entities to populate autoamtically.

I want ot make sure this functionality is accessible from Siri and is available in spotlight.  Impelment an app shortcut.

Rather than use application name in string, I use a special token.  Siri can substitute these in some way?  App shortcuts supports trigger phrases.  Recommend supporting the case where no parameter is provided so we prompt the user to provide the parameter value.

For optional parameters, we can manually prompt for value?

Short title, system image are important.

You can use parameters that are app enums, so they're known ahead of time.  Or use app entities to be fully dynamic.  Return a list of entities in your query.

* each time possible paramter values have changed, update them
* call upon a first app launch also

Remember, call this upon your app's first launch.  These won't work until the system has successfully fetched entities for the first time.

* spotlight top hits now shows app shortcuts
* available by searching app name, or app shortcut description

siri tips
* helps users discover your app shortcuts
* available in both swiftUI and uikit.  a number of styles, etc.
* pplaced contextually to screen contents

more resources
[[Dive into App Intents]]
[[Implement App Shortcuts with App Intents]]

# Great visual and voice experiences
app shortcuts can be seen in:* shortcuts app
spotlight top hits
automations

new APIs
* colors
* entity thumbnails
* short titles with symbols

up to 2 colors in app's info plist
consider adopting colors similar to your in-app's style, to bring your in-app experience into the system

each entity instance can have an optional thumbnail image.  Extension of `DisplayRepresentation` API.  As a URL, a data object, a named ubndle image resource, or a system image name.

short title and system image will be used to style your action.

in iOS 16, only recognize exactly as defined in sourcecode.  But people using your app may use different words/phrasing.  ex, 

> Summariez my groceries list with demo

vs 

> Tell me the summary of my groceries list with Demo

this now works.  On-device ML to allow similar phrases to also just work.  Powered by new semantic similarity index.

* no need to enter every variation of a phrase
* siri on-device ML broadens matching
* no change required - just rebuild with new SDK.
* opt out available, see build setting

synonyms API
* alows even more varied vocabulary with app intents
* supports synonymos for app entities and app enums

also works if siri prompts me for a list.  Synonyms are associated with each isntance of an entity, if they change let us know.

Now a new negative phrases API.  Filters incorrect phrases from invoking your app.  

New Xcode tool to speed up development.
Preview Flexible Matching in any locale
Only available on macOS 14

test my shortcuts by launching my app, etc.  New feature can be found in product -> app shortcuts preview.  Build app first.  So semantic similarity index can generate.

After building, can select app and enter phrases.  Can also switch languages.

Adopt string catalogs
Enables having different number of phrases for each locale
Only to apps targeting iOS 17 or later.

Create a new string catalog called app shortcuts.
string catalog keeps up with phrases you add/remove automatically.

If you've already adopted app shortcuts, migrate strings to string catalog.  Right click, migrate.  Use migration assistant.  See your phrases populate automatically.

Now you can add phrases to each locale without limitation.

* trigger phrases should be memorable and to the point
* Take advantage of app-name synonyms
* Use human-readable, interpolated strings for phrases
limitations:
* maximum of 10 app shortcuts
* max 1k trigger phrases.  All parameter combinations, etc.  Ensure any combination of parameters and phrases don't have too many possible values.
* Trigger phrases must contian app name (or synonym).  Refer to link.
* App intents that open when run are now shown in spotlight
# Extending across devices
apple watch.  Be aware of some limitations.
* app sohrtcuts must be defined in an installed watchOS app
* cannot run app shortcuts from paired iOS device

considerations
* no flexible matching on apple watch
* phrases must be psoken exactly
* supported on watchOS 9.2 and later

discoverable on apple watch.  Shortcuts app features available shortcuts

homepod
* supported on version 16.2 or later
* requiers app shortcuts on a companion ios app
* app shortcut may not launch the app

always return clear and concise dialog
take advantage of full and support dialog in IntentDialog

[[Design App Shortcuts]]

# Wrap up
* raise visibility and lower friction of your app
* more discoverable via spotlight
* siri flexible matching improves flexibility
* easy to develop and test with app shortcuts preview
[[Explore enhancements to App Intents]]
[[Design shortcuts for Spotlight]]

# resources
* https://developer.apple.com/documentation/AppIntents
* https://developer.apple.com/documentation/AppIntents/app-shortcuts
