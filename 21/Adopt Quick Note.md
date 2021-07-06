Uses an API that already exists, `NSUserActivity`.

# Quick Note
I can get back to my notes from these apps with quicknote suggestions.  
# How it works
NSUserActivity provides aw ay to capture the state of an app and put it to use later.  

Provide info about what is happening.  System gets the activities and sends them out to features.  

Quicknote piggybacks on this system.  In addition to handoff, spotlight, reminders, activities are sent to quicknote.  How links show up in menu.

* Must set one or more of `targetContentIdentifier`, `persistentIdentifier` or `webpageURL`.
* `targetcontentIdentifier` => used in state restoration
* `persistentIdentifier` => spotlight
* `webpageURL` => fallback during handoff.

Identifiers are
* Unique
* Global
* State

Identifiers enable Quick Note Suggestions.  

## Adopting
* Declare supported activities in info.plist
* Create and register activities
* Handle continuation

System uses info in the plist to determine if you're capable of handling useractivity object

Instead of registering hte activity by anaging current app activity, let system mange it for you.  Object becomes managed by UIKit and AppKit.

QuickNote launches your app with `scene(_ willcontinueUserActivityWithType)`.   Provides activity with `scene continue userActivity`.  If devices can't connect, you will get `didFailToContinue`.  Not invoked for QuickNote but good to implement.

Similar APIs for macOS.  

[[Adopting Handoff on iOS 8 and OS X - 14]]

## Other NSUserActivity features
* Handoff
* Siri shortcut donation
* Multi-window and state restoration
* Spotlight indexing
* Siri remind me about this

[[Spotlight indexing making the most of Search APIs - 16]]
[[Siri shortcut donation Introduction to Siri Shortcuts - 18]]



# Best practices
* Titles
* Identifiers
* Current activity
* Compatibility handling

## Titles
* Human-readable
* Descriptive
* User direct title

Use the title directly.
A few tips to help identifiers b eunique, global, and stable.

## Identifiers
* Avoid using device-specific data
* Avoid transient information
* Think long-term

Although URLs can be unique, they often contain transient info.

* Prefers targetcontentIdentifier and persistentIdentifier over webpageURL
* Use same identifier for `targetContentIdentifier` and `persistentIdentifier`
* Use `webpageURL` as a fallback

## Current activity
* Keep title and identifiers accurate
* Stay consistent with what's on screen
* Reusing instances not recommended

 When ther'es new content, create a new activity.
 
 * Attach to responders (vc or view)
 * `becomeCurrent()` `resignCurrent()` if needed.
 * Improve performance with `needsSave`.

Can be passed along with the activity, but there's overhead in updating every gesture.  Consider setting `needsSave` to true.  You will get a delegate callback here and can update properties on demand.  

## compatibility handling
* App has been updated
* Content doesn't exist anymore
	* Show an error message, redirect

# Wrap up
* Adopt NSUserActivity
* Update identifiers
* Use responders

