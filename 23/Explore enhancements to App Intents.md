Learn how you can help IT administrators get the tools they need to manage their organization's devices. Discover the latest changes to declarative device management, including software update management, additional asset types, status reporting for FileVault, and more.

# Widgets
Parameters.  Same system used to add support for siri and shortcuts to your app: intents.

#appintents 

ordered list of parameters that are included in corresponding intents.  Each parameter added to the intent is presentied as a row in widget configuration interface.

In the past, you had to declare your intents in xcode.  Now, in iOS 17, we have made it even simpler to define the schema of your widget configuration.  Use right in your widget extension code.

Direction of travel, etc.  Once I've finished defining the parameters, I'll need to provie dynamic options for each parameter type.  In the past, providing dynamic options for a parameter rquired creating a separate intents extension.  With app intents, I'll implement queries, etc.

[[Dive into App Intents]]

Support latest and previous OS
enable continued use of existing widgets
Remove intent definition

If you do plan to support previous versions, use new parameter and maintain sirikit definition file.  Add new parameters in there as well.

Previuosly-configured widgets automatically migrate
make sure to test!

[[Migrate custom intents to App Intents]]

Tappable buttons

SwiftUI button and toggle support App Intents
Shared execution code

[[Bring widgets to life]]
[[Dive into App Intents]]

## Dynamic options
Interface for providing available parameter values.
In some cases, you may want to show options that are only available when a certain condition based on value of another parameter is met.  ex, only route options for specific bus stop.

`@IntentParameterDependency`.  Read `AppItent` parameters from DynamicOptionsProvider

Works for widgets, shortcuts, focus filters

Array size.  ex, support only up to 3 routes.

New in iOS 17, can now use `When(widgetFamily:` to change config based on widget size.  Show only in large widgets, etc.

This year we added `RelativeIntentsManager`

can provide date information to suggest within the smart stack.

relevance 
[[Build widgets for the Smart Stack on Apple Watch]]

# Developer experience

In iOS 17, frameworks can expose app intents directly.  CAn now use `AppIntentsPackage` to recursively import dependency

Keep app intents adoption in frameworks
Simple AppIntentsPackage API
Shared code between app and extensions

App Shortcuts in extensions

Previously, had to define app shortcuts inside main bundle.  Now you can define app shortcuts in an app intents extension.  Great for performance, because you can optimize your app intents extension to come up faster.  Avoid bringing up UI, analytics, etc.  All these features rely on static metadata extraction.

Swift compiler offers information about types in your code.  Another tool when parsing this information... Metadata.appintents.  

Static extraction in xcode 15 is faster, more reliable, and works in more cases.  When building your app with xcode 15, if xcode is unable to statically extract something it expects, you will see error messages directly in xcode error.  With line numbers to fix the problem.

`ForegroundContinuableIntent` - inititally start in BG, but may need to continue in FB.

`needsToContinueInForegroundError` -> system stops performing app intent and asks the user to continue execution in fg.

continuation closure -> executed in main thread to update app state.  Here I use this closure to navigate my app to an error screen.

Use this error when you want to stop intent execution and require action to continue.  

`requestToContinueInForeground` -> optionally in fg, but not required.  

added support for apple pay to app intents.  Initiate an AP transaction directly from intent.  use applepay from perform is simple.  

# Shortcuts app integration

App intents are even more accessible, with live activities, swiftui apps, etc.

app intents can be used in many different ways.  Since app intents are now deeply integrated, important to ensure that we have good summaries, etc.

Write parameter summaries so that they read like a sentence.  System will determine the optimal visual presentation based on context.

* provide parameter summaries
* Use `isDiscoverable` as needed

Can now conform intents to `ProgressReportingIntent`.  This is surfaced in shortcuts.  Access progres and update it.  Progress will show up in the shortcuts app, use for long-running intents.

Also improved find actions.  Find notes, outputs, etc.

* Implemented using EntityPropertyQuery
* EnumerableEntityQuery
* Complete list of entities in `allEntities()`
* Simpler to implement than EntityPropertyQuery
* Optimized for a small number of entities.  Not suitable for large numbers of entities.  Continue using EntityPropertyQuery for these.

Result value name -> name the return type.  ex 'new reminder' if you make a reminder.  Parameter in shortcut action provides that name.  

You can also include an intent description for find actions that are geenrated with the queries.  Simply return the find intent description property within query types.

If you categorize actions with category name, display them under the desired category, etc.

# Wrap up
* app intents expose your app to people
* Build configurable, interactive widgets
* enhanced developer experience
* seamless shortcuts integration
[[Spotlight your app with App Shortcuts]]


# Resources
* https://developer.apple.com/documentation/devicemanagement
* http://github.com/apple/device-management
