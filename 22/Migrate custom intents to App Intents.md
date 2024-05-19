Learn how you can easily convert your existing custom intents to App Intents. We'll take you through the conversion of your intents to Swift and discuss how you can improve discoverability of your app features when you create App Shortcuts. To learn more about App Intents, watch "Implement App Shortcuts with App Intents" and "Dive into App Intents" from WWDC22.

# Why adopt App Intents

SiriKit Intents -> 2016.  Set of purpose-driven system intents that provide a complete user experience for common usecases.

Custom intents -> define your own intents for any usecase, bringing your app's usecase to siri, shortcuts, etc.

widgetkit -> custom intents for prediction.

App Intents -> new swift native framework.  Great framework to adopt because it's modern, powerful, and easy for users.

Modern: designed natively for Swift.  Less code required.  No intent definition files or code generation.  SwiftUI snippets.

Powerful: Entities/Queries enable more expressive use cases.  App intents can run directly in process.  New opportunities for customizing UX.

Easy: App Shortcuts are enabled automatically.  People can use your intents with no setup required  New discoverability features.

You'll need to upgrade custom intents to App Intents.  SiriKit are still fully supported, so if you're doing that stuff or widgetkit, leave as is.

[[Dive into App Intents]]
[[Implement App Shortcuts with App Intents]]
# Migration overview

Convert to App Intents with single click
Support latest and previous OS
Enable continued use of existing shortcuts

Press the button!  
Xcode produces swift files with app intents code
populate the code by refactoring

mapping between legacy intents and app intents, system automatically takes care of mapping old intents to new ones.  Adopt `CustomIntentMigratedAppIntent` protocol.  You provide intent class name.  In most cases, you shouldn't have to provide this yourself.  Use the convert to app intent button.

Because of this capability, you can support both iOS 15 and iOS 16 with the same app.

Include both legacy intents and app intents in your app.  to maximize codesharing, share business logic.  Shortcuts will automatically deduplicate intents.

On iOS 15, shortcuts will only show legacy intent implementation.  On 16, we only show app intents.  Once you switch to 16, delete legacy intent handlers.

People have existing shortcuts using your legacy custom intents.  When people use those existing shortcuts, the intents will resolve to the new App intents (if you use compatibility protocol).
Intent schemas must be compatible:
* same parameter names
* and types
but ok if
* add or remove parameters
* make existing parameters non-optional

# Converting intents to App Intents
steps:
1.  Migrate intent definition to app intents
2. Migrate intent handling code
## definition file

demo

## code

Note that we have a placeholder for the perform method with a TODO:

In old framework you provided definition file and intent handling code.  in new framework, everything is code.  Parameters were migrated.  But you need to do resolve, confirm, and handle, into the perform thing.

Dynamic options must be refactored into entities/queries.

## migrating resolve

When migrating resolve, take advantage of automatic resolution where possible
otherwise refactor resolution into `perform()`.

Just make type non-optional.  App Intents will automatically ask the user for a value if it's empty, using suggestedEntities.

Can specify `requestValueDialog`.  Don't need any reoslution code anymore.

For more complex resolution
* move resolve implementation to top of perform.
* use new APIs to ask user for clarification

`CustomLocalizedStringResourceConvertible` for our error.

If you used to have code that returns a needs value result dynamically, we can throw a `needsValueError`.  

Can use `requestValue`.  `requestDisambiguation` and `requestConfirmation`.

## confirm

Opportunity to do additional validation and supply information to ask the user to confirm that they want to proceed.

Move validation logic to perform and throw errors when needed
If your intent requests user confirmation... use `requestConfirmation`.  Cancel throws an error.

## Handle
Refactoring handle, simply move to perform.  
Remove validation code that is no longer necessary

Return an intent result.  ex, dialog to speak, snippet for showing, etc.

need `ProvidesDialog` or `ShowsSnippet` in return type?

Fill in query or DynamicOptionsProvider.  For a query, need to implement methods for returning entities by identifier, string, or suggestedEntities.

for options, just return all the options in results method.

# Recommendations
* Adopt App Shortcuts for ease of use
* Use Siri tips in place of Add to Siri buttons on iOS 16 and later.
* Test customer upgrade path
* Localize

AppShortcuts.strings.  Replace variables with `${foo}`.

# Wrap up
* upgrade your custom intents to App Intents to support iOS 16's new features
* Add App Shortcuts so people can discover your app's features
* [[Design App Shortcuts]]
