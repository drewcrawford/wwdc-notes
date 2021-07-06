# Session
* Text
* Audio
* Video
* Group mangement
* System-wide

# Platform
Unified experience across iOS, iPadOS, macOS, apple tv.

Great audio to bluetooth including airpod.

# Playback
The moment when your users decide what content to spend their time on.  We want every play button to work.

Your play buttons can now start these experiences whenever a new conversation is active.

Syncing at the platform level.  New sync protocol.  

**We won't retransmit media in any way.**  Never compromises content quality.

Smart volume will automatically duck audio and bring it back up when appropriate.

PIP

# Content
On a facetime call, they expect to be able to go into an app and share the content.  SharePlay.

# What are Group Activities?
Users can navigate to your app and will be notified that an app supports shareplay.

Create an object that implements the Group Activity protocol.

Call `prepareForActivation`.  Once users have joined, video will be kept in sync with the group.

Once users are done, they can choose to end the activity for themselves or whole group.


# Concepts and architecture

Swift-native framework.

* `GroupActivity`
* `GroupSession`

## GroupActivity
* Use to define the shared experience
* Holds application-specific information about the activity

For shared audio, video playback it holds content URL.  Or application may provide a custom shared experience.

Pulls information about what users are drawing


## GroupSession
* Representation of the group
* Access to participants
* Additional API for syncing data across devices
* e2e encrypted

Not meant for exchanging large amounts of data.  This channel is used by AVFoundation to keep content in sync by exchanging play/pause/seek commands

# Initiating an activity

1.  Create an object that conforms to `GroupActivity` protocol.
2.  start

# Observing a session
Your application needs to iterate over incoming sessions via an async sequence.  Then, when there's a session, application gets handed the group session object.

Same flow whether hosting or clienting

[[Coordinate media playback with group activities]]

# Joining a session
Could mean loading assets, etc.

For avplayer,

1.  GroupSession.Sessions
2.  GroupSession
3.  AVPlaybackCoordinator
4.  AVPlayer / your player

Can use any custom video playback and still get support for syncing.

Call `.join` on the group session.  At this pont, your application is ready to sync data and let your users have shared experience.

Can use this channel to exchange data to keep your users in sync.  Used by AVFoundation to keep media playback in sync.


Not to be used to exchange large amounts of data. 

# Posting events
[[Build custom experiences with Group Activities]]

# Wrap up
* Cross platform Swift API to create shared experiences over FaceTime
* AVFoundation integration for shared media experiences
* Supports playback sync over web on macOS.

https://developer.apple.com/documentation/avfoundation/media_playback_and_selection/supporting_coordinated_media_playback
https://developer.apple.com/documentation/GroupActivities