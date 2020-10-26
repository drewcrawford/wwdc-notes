#sirikit 

# Prerequisites
* already support SiriKit media intents on your iOS app
	* developer.apple.com/documentation/sirikit/media
* Apply for SiriKit media intents on HomePod
	* developer.apple.com/siri
* Access specifications for cloud extension APIs
* Register your service
* Implement OAuth and Configuration Web Services

Need to provide
* service name
* public signing key
* static url for your service's icon
* bundle IDs of your apps using the service

Only devices with the profile will be able to test your service.

## Oauth 2.0 endpoint
* a server capable of uauth 2 client credentials flow
* Unique clientID and clientsecret material for each account
* issuing renewal tokens recommended, to avoid common re-use, without this...
* clientID and clientsecret can be long-lived

Your app must be able to spply the credentials from the service and provide to iOS.

## cloud extension configuration resource

* Implement configuration resource endpoint(s) as defined in specification
* Potentially multiple configuration resource definitions
* Body encoded using JOSE and properly signed
* This resource must use HTTP cache control

Your app provides this URL to add service to the home.

# How to configure a test home
* dedicated test home entry in the apple home app recommended.
* Multiple test iCloud accounts
* Ensure same iCloud accounts are being used accross iOS devices

# set up your HomePod
Usually install tvOS Developer Beta release on HomePod.  Note that you can't use regular tvOS release until you're ready to submit your app.
Install your service's development profile on homepod in the apple home pap
repeat for multiple test devices
When upding tvos, may need to reinstall profile.

1.  Select test home on home app
2.  Open profile on iOS and install


# Set up your iOS device
* Install the latest iOS developer beta
* Install your developer profile via settings
* Use matching beta releases for both iOS and tvOS test devices
* Optionally, repeat for multiple test iOS devices (iPhone, iPad)

# Adopt MediaSetup framework
* implement retrieving oauth credentials
* Implement choosing a configuration resource URL for the account
* Add entitlement for mediasetup use.
* Add onboarding UI flow in your iOS app using MSSetupSession

# Intent handling in the cloud
Typically, intents are handled by app extension.
However, in this case we use server-side APIs.

 * web api for intent handling interactions
 * object definitions closely follow APIs on device
 * Each SiriKit media intent type can be configured

## supported protocols
* play media
* add media
* update media affinity

Search is not provided since we can't display results.

## Details about flows
Mostly what you would expect

## Example
We provide a deadline.  Providing a timely response is important.
Consider how to break up intent results.

Provide a `metrics` data to determine how long it took your server to deliver results.  Apple uses this to understand network delay.

# Media queues in the cloud
In certain situations, the homepod that resolves the intent may not be the same homepod that is playing your content.

## Providing content via queues

Playback is defined by a queue of content items
Created or modified by a user activity resulting from intent
Your service provides the queue all at once or in segments

internet radio vs album vs live streaming.

content in a queue can be described with an identifier, url, attributes.
content selection can tak eany number of algorithmic...
Queues support all these usecases.

## Media queue idioms
Array of content items.  Static content idiom.  Queue contains a finite, non-changing list of content.

Usecases include on-demand music request (e.g. album/playlist)
or livestream (wrapper for single content that streams indefinitely)

queues can be split into segments that internally link to next/prev.  HomePod will retrieve as service defines them, and a particular segment can be dynamically generated.

e.g., consider the case that you have a playlist containing an ad.  Each ad is different, so we dynamically generate that one.

Fully-dynamic: if you generate a queue of things that your users might like, changing behavior might change the queue.

## Media queue management
* new queues result from completed `PlayMediaIntentHandling` requests.
* Each queue segment can link toa next/prev segment
* A playPointer can be specified to resume prior playback
* Preroll fetch made for next queue segment before the end of content

Note that the device version may be different between devices in the home.
Don't rely on linking the intent results with the queue via identifier, use userActivity instead.

### Inserting content int oa queue
Include an `insertPointer` object definiing where content is added
Avoids raceconditions since the pointer... time between request and response
### queue management details
* Use fully dynamic idiom to define experience with most flexibility
* Maintain consistency in queue history
* Implement repeat modes and shuffling behavior in your service.  You have to implement these yourself since it requires full knowledge of the queue extent
* Use content URL templating to react with context
* If necessary, update queues via reporting events

## controlling playback
Not all content in a queue is the same.  e.g., internet radio typically only supports forward motion.  Advertisement may not allow skip.  Often, fully-dynamic is the only way to support these requirements.

* Define PlayMediaControl objects in your queue implementation.
* Choose a predefined scheme to get default and specific behaviors
* Set the control attribute on each content item

### example controls definition
Definition for `default` and `ad`.  Provide different schemes.

# Sessions and reporting
## reporting user activity
### implement update activity endpoint
Implement to get status reportin fom the Cloud Media Player.
Receives reports for transitions and media control events
Reporting can inform dynamic queue content sselection or provide new queues
Request are made asynchronously from content retrieval and transitions
### example
### advanced
* add playback interval reporting via `PLayMediaControl` definitions
* Enable `MPRemovecommandCenter` commands like bookmark, dislikeTrack, likeTrack
	* Note these are different than the intents
* Update User Activity objects during playback
## using session context
* Only available if the homepod is already palying from your service.
* Includes content identifier, queue identifier, user activity persistentidentifier
* Can replace complex reporting implementations for some services

## adhering to constraints
Reflects HomePod settings and realities of shared listener requests
Constrains may differ on different homepods in a home
Processing of requests must follow the constrains presented.
May be affected by the detection of which person interacted
Constraints should only further constrain any limits in service account settings.  If your service has account-specific restrictions, combine them to produce an effective situation.

Specific methods to respond if the only results are explicit.
`maximumQueueSegmentItemCount` per-device.

# next steps
* review the spec

visit developer.apple.com/siri

