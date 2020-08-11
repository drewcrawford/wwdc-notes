#sirikit
# Overview
* intents
* intentsUI

System intents, custom intents.

## Intent handling lifecycle
Your handler has 10s to complete request.
Starts as soon as the user request connects to your extension.  If your extension is not yet running, it will be launched.  Amount of time it takes depends on how long it takes to load the frameworks.
Plus `+load` and static initializers, like `main()`.
Then business logic.
So it's important to optimize for lanch time, by only linking frameworks you actually need.

Siri interactions are intended to be quick – avoid linking unneded frameworks!

## Intents extension
* can link a subst of app frameworks
* separate process



# What's new in SiriKit
## in-app intent handling
In iOS 14, we allow in-app intent handling.  Which should you use?

Good use-cases for in-app intent handling:
* starting or controlling media playback, workouts
	* more efficient to do entirely in your app
* Interacting with on-screen UI
* Performing intents which need a lot of memory. (photo/video)
* Handling intents with code that cannot be shared with an extension.

Evaluate which events need to be handled in extension vs app.

How to implement in-app ahndling?
Need to check "Supports multiple windows" and supports UIScene.  It will be launched **without any UIScene objects connected**.

List intents you handle in the "supported intents" section of your app's target.

Implement `AppDelegate.application(application:,handleFor intent:)`
Return objct that implements the protocol for the intent class you were passed an instance of.
Return `.continueInApp` if you're in the background before manipulating UI you expect the user to see.

## demo
Have users advance to next step in my recipe app.  Users will be interacting with the content on-screen.

## Intents extensions
* can be more lightweight and faster to launch
* optimize for launch time

## in-app intent handling
* consider implementing an intents extensionf irst
* be mindful about the numbe rof frameworks that your app links

[[Introducing Multiple Windows on iPad - 19]]
[[Architecting Your App for Multiple Windows - 19]]

## Rich Disambiguation
Earlier, we introduced diambiguation.  User prompted to pick from a list of values.
This year, we're allowing you to include subtitles/images in those.  Just provide subtitle as a string, and image as INImage.

Can also provide dynamic options when they configure your intent in the shortcut tab.

Now supports pagination, so you can specify number of items.  

### Implementation
Expand "Siri dialog" for the parameter.
Specify the "paginate every".  As well as "Subsequent introduction" string.
Can also specify the "Disambiguation Introduction" dialog.

## Dynamic Search

Last year, we introduced dynamic options.  Can provide a set of values for parameters dynamically.  

This year, we're expanding the API to include the search term provided by the user.  (autocomplete results).  If you check the "provides..." we will pass the search term to you and it's called as they type.

**Only adopt for searching in large catalogs, because the shortcuts app supports filtering by default.**

Can also group items in sections now.

## Configurable vs resolvable parameters

This year, you can mark a parameter as "configurable" and "resolvable" separately.  "Siri will not resolve parameters that are not marked as resolvable".  Not 100% I understand this.

# Get the most out of your intents

* Custom intents deprecation -> It is now possible to deprecate custom intents using xcode 12.  Select the intent and then choose "deprecated" in the righthand pane.  Users get a warning that this deprecated.
* Custom intent classnames.  Actual class is codegenerated for you, based on the typename.  So if your app uses... custom class, it's not the right way to specify enums.  Now can specify "Class prefix" or "Custom class" like you can in IB.
* Managing multiple intent defintiion and Intents UI extension.  Sometimes you need custom UI for confirm/handle case, but not all.  For some custom intents, you might not want to display any custom UI.  To do this, create separate definition files for UI vs non-ui intents.  Then in righthand pane, remove the UI target for the non-ui definition file.
* "Intent Class Code generation launguage" buil setting.  By default, xcode will automatically decide which language to generate based on sourcefiles in the target.  But you can now customize this behavior.

