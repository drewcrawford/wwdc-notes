apple is good and stuff

buzzwords
* secure environment
* stable devices/apps
* operational continuity
* great user experience


# Testing pre-release software
"The truth is, we can't possibly replicate all the education and enterprise environments out there."

Public beta is geared toward reporting livability or general use type issues.

Developer program for developers.

AppleSeed for IT.

# AppleSeed for IT
Seed program for IT Professionals in enterprise and education.  Apple deploys a variety of software for you to test.
In-depth documentation and release notes for you

* test plans -> for you
* surveys -> for us

Provide your unique feedback
Identify deployment blockers
Report regressions

## get started

Create a Managed Apple ID
Log in to enroll at https://appleseed.apple.com
Configure your devices

Encourage collaboration with other participants and teammates.
Field engineering
AppleCare

Submit all issues you encounter through FeedbackAssistant.  
You can request testing assistance from an AppleCare account management etc.
Can notify them about deployment blocking issues

Filing feedback for your organization


# Filing feedback for your organization
* file immediately after the issue occurs
* gather logs and note the time
* steps to reproduce
* screen recordings or screen shots

## feedback assistant
collects logs/diagnostics, keep track of bugs throughout apple, etc.

Organizations want to share info about bugs that affect more than one person.  Thsi year we're introducing Teams.  Members of an organization can work together.  Configured by ABM/ASM or ASC (developers).
See feedback submitted by your peers
Contribute to discussions with Apple
Reassign feedback to team members

Often, we need diagnostics from multiple devices.  So we added multi-device diagnostics.
Initiate feedback from an iPhone or iPad, you can collect logs from multiple devices
Must be signed into iCloud

### demo
In iOS 14, you don't have to wait for diagnostics to complete to submit the feedback.  FB assistant figures it out

Move feedback from personal space to team space

### configuring your team
Ensure that roles you want to participate have "participate in appleseed for IT" In ABM/ASM.

### recap
* Testing helps you prepare
* AppleSeed for IT program
* Filing feedback for your organization

# Managing software updates
Critical for IT / sysadmins to be engaged in prelease software.  Ensures you/organization will have a smooth transition.  Ideally, orgs should be deploying updates as they're released.

However, sometimes you need more time.

* control over updating Apple devices
* Update compatibility with your organization
* Consistent deployment across devices
* Contain critical improvements for stability, security, etc.

Starting software update via MDM
* management command to update devices
* choose to download only or download and install the update
* Requires supervision for this to work
* iPhone and iPad requires passcode to be entered by user

## iOS, iPadOS, tvOS

* MDM restrictions that defers OTA software updates
* default delay is 30 days
* can specify 1-90 days
* After delay expires, the next update in the deferral window is evaluated
* No downgrades or rollbacks

## macOS
* schedule a scan
* Fetch the list of avilable updates
* get status of an update in progress
* schedule updates for installation
* requires supervision
* control over user settings in software update preferences

* deferred updates are transparent to the user.  
* mac does not need to be supervised
* deferral window determined by date

We want IT and enterprise engaged in testing.
* Support for deferring software updates during seeding in macOS Big Sur
* Support for deferring major releases was introduced in macOS Catalina 10.15.4

Unification of installation technologies
Snapshot based updates
Cryptographically sealed system volume
Remotely driven updates

### deprecations
* custom catalog support has been removed in macOS Big Sur
* No longer possible to ignore updates indefinitely
	* Still supported in these releases if the device is supervised
		* macOS Catalina 10.15.
		* macOS Mojave Security Update later this year
	* We advise using update deferral instead

# wrap up
* participate in and join appleseed for IT
* Work with your extended Apple teams
* Utilize Feedback Assistant extensively
* Manage software updates in your organization

[[20/What's new in managing apple devices]]

