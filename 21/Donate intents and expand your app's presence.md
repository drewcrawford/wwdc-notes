#sirikit 

# What are intent donations?
Shortcuts let you expose the capabilities of your app as actions.  Enable people to use your app's features in new ways and places.

Every action should be
* Accelerative
* Repeatable
* Stateless

[[Design great actions for Shortcuts, Siri, and Suggestions]]

* NSUserActivity
* Intents

[[Introduction to Siri Shortcuts - 18]]

## Built-in
* Messaging
* reservations
* media
* many more

Also custom intents.  Use builtin if possible, custom otherwise.

## Intent donation
Act of telling the system when a person performs an action in your app.  System will store donations and use them to expand capabilities and presence.

Show up throughout the system, suggestions, shortcuts, focus, smart stack, siri.


# The life of an intent donation
Check weather.  

1.  Create intent definition file
2.  Define intent.  For this we use a custom intent.
	1.  Configure your options, like "is eligible for widgets" etc
	2.  Parameters
		1.  Custom types
			1.  Dynamic options, etc.
	3.  Supported combinations (of parameters)
	4.  Need to donate intent to the system when someone checks the weather.  

Can delete donations

All ML and intelligence is performed on-device in a privacy-preserving manner.  

System leverages intent donations that match configuration.  

If they already had weather widget, would have torated instead.

[[Add configuration and intelligence to your widgets]]

Power on-device intelligence.  Perform key capabilities with one tap, as well as shortcuts.  Determine what actions get suggested in shortcuts app gallery.  

INSendMessage is surfaced in sharing suggestions.  Also uses this for focus.  

Opening maps.  

Now playing UI.

## INUpcomingMediaManager
* Great for periodical content like podcasts, tv shows, or movies
* Privde a list of media intents the user hasn't listened to or watched

## INRelevantShortcut
* Expose to Siri Watch fAce

[[Siri Shortcuts on the Siri Watch Face - 18]]

* new widgetKind property.  
* Hints to system when to suggest a widget

[[Add intelligence to your widgets]]


# Structuring your donations
* Size of coffee, coffee item

1.  Donation comes in
2.  Date parameter has a new value
3.  Treats donations differently over time
4.  Can't find pattern

When the supported combination is all 3, system uses all 3 to find prediction.  Prevented system from finding a pattern.

Instead defne only item,size.  The system knows to only use item and size when comparing intent donations.

## What makes a great donation?
* Likely to be repeated
* Intent payload is concistent across donations
* Don't include timestamps

# Wrap up
Donate intents to integrate with the system
Expand your presence

* https://developer.apple.com/documentation/sirikit/donating_shortcuts


