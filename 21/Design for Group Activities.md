# Group Activities and SharePlay
Facetime is now open to you!

* Watch movie with a distant relative
* Shoot perfect 3-pointer
* Share photos
* Learn new language

# Discover shared experiences
* App UI => Which parts are relevant?
	* By default,secre entry fields and hidden UIKit views are disabled.
	* Use APIs to further restrict during screensharing sessions.
	* Audio is shared automatically, but
		* Protected content will not be shared

## Coordinated media playback
* time synced
* Handle playback controls
* Smart volume

[[Coordinate media playback with group activities]]

## Custom UI
* Use the message passing API
* Draw custom views

[[Build custom experiences with Group Activities]]


# Enhance for sharing
?
# Add context
When your app is first launched, you're shown if it supports shareplay.  If your app contains video content, you have an opportunity to communicate.

Tap the shareplay button and everyone's show will start.

* Let participants know about what's going on
* title
* subtitle
* Image preview

In this example, importantt o make these titles and subtitles meaningful.



# Automate
Whenever someone's interacting with their phone, they're multitasking.  Make sure the interaction is as easy as possible.  Automate as much as you can.

* Start playback without your app moving to the foreground.
* Continue playback when your app is in the background.

[[What's new in AVKit]]
[[Delivering intuitive media playback with AVKit - 19]]

* Login flows and subscriptions requiring interaction
* use group activities API to request foreground presentation

Question every interaction and see if it's necessary, don't hold up your friends.

# Wrap up
* Discover experiences
* enhance for sharing
* add context
* automate

* https://developer.apple.com/documentation/GroupActivities
* 