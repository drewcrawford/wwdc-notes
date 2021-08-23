#watchOS 

New opportunities for features like Family Setup for non-iPhone users.

# Always-On Display
Now Available to your apps.

Series 5 and 6.  In watchOS 7, showed your app's UI blurred with time overlayed.

When rebuilding with 8 sdk, your app's UI will now be shown in a dimmed state instead and is immediately interactive if tapped.

* Reduces the overall brightness of the display
* SwiftUI environment property called `isLuminanceReduced` you can respond to.
* Hide information that should be private.

Principles.
* Transition should feel seamless.  Don't drastically reorganize.
* Dim non-essential information and elements.  e.g. secondary text, images, fills, further dim these yourself.
* Removes the background of list items and further dims the secondary information.  
* Large elements might be reduced to be stroke or dimmed color.  Maintaining UI's grounding but improving constrast.
* Redact or remove sensitive information.Account balances, health data, etc.

[[Principles of great widgets]]

* Reset animations to the first frame of their loop or resolve them to a rested state.

* Active session (workout etc) => Update up to once per second. Remove subsecond incrementing.
* Outside of sessions
	* Updates up to once per minute
	* Assume your app is visible longer than two minutes

## TimelineView
Constructs a view depending on a date.  TimelineSchedule.

Schedules:
* Every minute, aligned to system clock
* Periodic.  Interval
* Explicit => specific times
* Custom => Timer.  Once per minute, and then once per second in another phase.  Note that if you're not in an active session, and you say you need short updates, we'll try, but it's not guaranteed.
* Animation

[[21/What's new in SwiftUI]]
[[Add rich graphics to your SwiftUI app]]

# HealthKit data
* Background delivery is similar to iOS
* Apps receive results up to once per hour
* Four per hour when complications are active on watch face
* All received results count against background app refresh budget
	* Can impact background app refresh wakes for other reasons

Update frequency
* IMmediate for critical data types.  Fall events, low blood oxygen, heart rate, etc.
	* See developer docs
* Hourly or longer for others
* Authorization sheet has additional disclosures


# Bluetooth
In watchOS 8, we're taking another step forward and allowing devices to connect during background app refresh that complications get on the active watch face.

## Bluetooth connection from complications
Complications can display updated information
Up to four opportunities per hour
First time setup must take place in app
Connect and process within short period of time
New expiration handler

[[Connect bluetooth devices to Apple Watch]]


# Location
## Region-based notification.
Similar to iOS
Static notification.  Tap a button to see full notification.

So you can deliver *pre-created* notifications
"When in use" permission is required for you to use this.
Limit regions to important POIs nearby
Power-efficient regions are ~200m

## Location button
One-time location auth without going through authorization prompts each time.

Behind the scenes, it acts like an allow-once authorization.  Location button is an easy way to gain trust by giving people more control of when they want to share their location.

[[Meet the Location Button]]

Always-On altimeter
# Enhancements
* Respiratory rate
* AssistiveTouch
* Large accessibility text size

[[Create accessible experiences for watchOS]]

* Unit testing and UI testing (xcode 12.5)
	* New accessibility features mean you can be more accessible
* Large titles
* Text input
* Searchable API

[[Craft search experiences in SwiftUI]]

* Swipe actions on List
* Button roles
	* Additional haptic when tapped for prominence
* Canvas

# Next steps
* TimelineView
* Complications
* Notifications
* Next level capabilities

* https://developer.apple.com/documentation/watchkit/updating_watchos_apps_with_timelines
* https://developer.apple.com/documentation/watchkit/designing_your_app_for_the_always_on_state
* https://developer.apple.com/documentation/usernotifications/unlocationnotificationtrigger




