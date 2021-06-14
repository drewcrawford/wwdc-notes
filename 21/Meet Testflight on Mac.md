TestFlight lets you distribute meta builds and collect valuable feedback.  Available on iOS/tvOS, now bringing it to macOS in **fall 2021**. #testflight


TF Will be available on MAS.

* Install beta apps
* Automatic updates
* Share feedback
* Native mac apps

You can recruit testers by sending an email invite or sharing a public link.  Once they accept the invite, they come a tester.

Once a beta app is installed, it can be launched from testflight.  It can also be launched every other way, e.g. dock etc.

We show a yellow dot in the app name.  Testers can configure automatic updates to have the latest build installed automatically.  This ensures they're testing the latest build of the app.

While testing a beta version, your testers can send feedback about issues they experience or make suggestions.  Can send feedback directly from the app by taking a screenshot and attaching to the feedback form.

Can be viewed in ASC under the feedback session.

Can download crash logs and see feedbacks under ASC crashes / feedback session?  Xcode organizer?

# Native mac app
 * Require provisioning profile
 * Included by automatically manage signing
 * Explicitly include for manualy manage signing
	 * see docs

Builds will be displayed under macOS
Create groups to manage testers, etc.  Similar to iOS or tvOS.
View the number of invited testers, installations, sessions, crashes, feedback, etc.  Aggregated across all testers.

ASC feedback can filter by macOS.  Can further filter by selecting a specific mac device or version.

iOS app on apple silicon mac.


# iOS App on apple silicon mac
Each group control.  "Test iPhone and iPad apps on Apple Silicon Macs".  Disable to make them unavailable on mac.  More flexibility to control who can test iOS apps on macs.

iOS builds are displayed like today.  

Can see crashes, screenshots, etc.

# Improved internal group management

Internal testing has always made testing convenient.  This year we made new improvements to internal group management, applicable to all platforms.

CAn now create multiple internal groups similar to external groups.
Configure distribution of builds
feedback settings

ex, dev vs QA.  

dev: "automatic'
QA: "manual" only specific builds.

How to set up?
Create groups for each team by clicking on the plus button for internal testing.  Give your group a name.

dev: "Enable automatic distribution"
QA: uncheck

You ahve the option to enable or disable feedback per-group.

* ASC API
* ASC on iOS


# Built-in xcode cloud

We've integrated exciting xcode cloud features.

Seamless experience to automatically build, test, and distribute apps.

[[Meet Xcode Cloud]]

Managed build distribution
Build Groups

For internal testers, build groups display makes it easier to find builds based on current branch name.

Filter feedback based on build group.

# Wrap up
"Coming soon"
IMproved internal group management
Built-in xcode cloud features #xcodecloud 
