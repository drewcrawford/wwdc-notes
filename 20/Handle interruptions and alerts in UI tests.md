#xctest 

# What is a UI interruption?
Any element that unexpectedly blocks access to another element that our tests wants to interact with

Commonly banners, alerts, or windows.  Interruptions are unexpected, or at least not detemrinistic.

Apeparance of an alert in response to a test action is not an interruption.
By definition, there is no way to handle interruptions deterministically or predict them

We now have a stack of interruption handlers.

## Built-in
* alerts
	* Prefers cancel button
	* Falls back to 'default' button
* banner notifications (new)
	* swipe to dismiss banners
* User ppermission dialogs (macos) - chooses 'don't allow'
* bluetooth setup window - closes window

## Demo
`addUIInterruptionMonitor`

## Not all alerts are UI interruptions
Some alerts appear explicitly in response to a given user action

In the demo, the test did not explicitly trigger the alert's apeparance.

## Work with protected resource

User permission is system-wide state
Testing scenarios
* How does app responsd to permission granted?
* How about permission denied?

[[better apps through better privacy - 18]]

In a point reelase, we introduced `resetAuthorizationStatus`.  This allows you to retest.

Note taht resetting authorization status may kill your app.  Also alerts come from system, so may need to adjust queries (I guess can't query with app at the root??)


