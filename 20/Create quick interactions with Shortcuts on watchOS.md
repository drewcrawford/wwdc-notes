#watchos 

# last year
Last year
* added ability for shortcuts to have parameters
* Run shortcuts from watch face

# What's new in Shortcuts on Apple Watch
New shortcuts app on watchOS 7
Run shortcuts with a single tap.
Shortcuts from iOS work just as well on watchOS.

Answer questions inline.
Support complications for new shortcuts app on watchOS.
Also set up complications that launch a specific shortcut directly.

Your app's shortcuts can be available on the watch face.


# Optimizing your shorcuts experience on watchOS
Quick recap of shortcuts.

Use SiriKit and NSUserActivity just like iOS.

[[Introduction to Siri Shortcuts - 18]]
[[Introducing SiriKit - 16]]

## How watch is different
Can configure shortcuts on iOS, and put them on watch collection and expect them to run there.

For shortcuts that are `NSUserActivity`, we need to open the app to handle the activity.  But we can't open the app if it's not installed on the watch, and in that case attempting to run such shortcut would result in an error.

Intent-based shortcuts: handled by intents extensions.  If the extension is installed on the watch, we hand it over.  If not, we will try to run remotely on the paired phone.  "Remote execution."  Note that is in contract to `NSUserActivity` stuff, which can only be run directly on watch.

But if the intents extension tells us that we need to continue in the app, just like the `NSUserActivity` case, we error.  Otherwise, the shortcut is executed remotely on the phone.

Design your shortcuts to avoid these error cases.

## remote execution
If intent is supported natively on the watch, we launch the local intents extension and process it there.

Similarly, user activity-based shortcuts will be opened in the watchOS app, if it's installed.

If however we have to handle the intent over to the phone, we introduce an additional hop into the process.  This inevitably increases the time to perform the task.  Best experience is to handle directly on watch.

## supporting shortcuts on watch
Best approach is to have a watchOS app.  Support same user activity or intents as your iOS app.  Consider building one.

If for some reason this isn't an option, ensure that your shortcuts work well with remote execution.

## remote exectuion
* only supported for intent-based shortcuts, not NSUserActivity
	* NSUserActivity require the app to be open in order to run, and opening the app on another device would be a very jarring experience
* only supported for intents that run without opening the app.
	* `.continueInApp` cannot be returned during confirm or handle steps
	* 'support sbackground execution' checked in Suggestions section, for all parameter combos

## implement shortcuts using intents
For all new shortcuts adopters, we recommend *Intents* because it provides richer and more flexible APIs than `NSUserActivity`.  Parameters, run in background, etc.

Make sure to compile your intents extension not only for iOS, but also for watchOS.  This usually means creating an additional target.

Ideally, try to support same intents on watchOS as on iOS.  

## Build for voice
No Intents UI extension support on watch
Carefully constructed dialogs are important

Custom responses are very flexible.  Provide any info that's relevant or helpful.

[[Designing Great Shortcuts - 19]]
[[Building for Voice with Siri Shortcuts - 18]]

## Siri watch face
Expose relevant shortcuts on Siri watch face
Use the `INRelevantShortcut` and `INRelevantShortcutStore` APIs
[[Siri Shortcuts on the Siri Watch Face - 18]]

# Recap
* Build a watch app or support remote execution
* Implement your shortcuts as intents over `NSUserActivity`
* Compile intents extension for watchOS
* Build a great voice experience
* Offer your shortcuts on Siri watch face



