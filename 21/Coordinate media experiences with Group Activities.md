# What is the GroupActivities API?
* Swift fraemwork
* Build shared experiences across devices
* Coordinated media playback
* iOS, iPadOS, macOS, tvOS

# GroupActivities
```swift
protocol GroupActivity: Codable {

    /// An identifier so the system knows how to reference this activity
    static var activityIdentifier: String { get }

    /// Information that the system uses to show this activity, such as title and a preview image
    var metadata: GroupActivityMetadata { get async }
}
```

Single piece of content like a movie or song.  Include any properties you need to setup your experience.  e.x., URL for video.

[[Meet asyncawait in Swift]]

1.  Define groupactivity
2.  call `.activate()`
3.  Shares the activity to the group, creates a GroupSession
4.  Local/remote receive thei session
5.  Launches app on remote device

How do you know you're in a facetime clal?
How does a user decide to watch content locally?

`prepareForActivation`.  

```swift
func playButtonTapped() {
    let activity = MovieWatchingActivity(movie: movie)
            
    Task {
        switch await activity.prepareForActivation() {
        case .activationDisabled:
            // Playback coordination isn't active. Queue movie
            // for local playback.
            self.enqueuedMovie = movie
        case .activationPreferred:
            // Activate the activity. The system enqueues the movie
            // when the activity starts.
            activity.activate()
        case .cancelled:
            // The user cancelled the operation. Nothing to perform.
            break
        default:
            break
        }
    }
}
```

Instead, we replace playback with `prepareForActivation`.  

How to sync the correct video?

`GroupSession<GroupActivity`
`GroupActivity.sessions()`

1.  Both devices receive groupsession
2.  Setup for playback
3.  Join groupsession

Session provides state about the session, latest activity, connection state, active participants, etc.  used to synchronize playback.

AsyncSequence delivers group session to your app.  Apps never create GroupSession objects directly.

Both local and remote participants receive group sessions this way.

[[Meet asyncawait in Swift]]

```swift
// Receiving a GroupSession from the GroupSession AsyncSequence

func listenForGroupSession() {
    Task {
        for await session in MovieWatchingActivity.sessions() {
            ...
        }
    }
}
```

## Playback synchronization
`AVPlaybackCoordinator`
Attach the GroupSession tot he coordinator to synchronize playback

```swift
let player = AVPlayer()

...

func listenForGroupSession() {
    Task {
        for await groupSession in MovieWatchingActivity.sessions() {
            
            // Verify content is available, prepare for playback to begin

            player.playbackCoordinator.coordinateWithSession(groupSession)

            ...
        }
    }
}
```

Framework handles all intricacies of playback etc.

INitially, `GroupSession` is not connected
Calling `groupSession.join()` connects to the session
One joined, playback synchronization will begin

`groupSession.leave()`
Local participnats leaves the session
Session continues for remaining participants

vs end, ends the whole call.

## Advanced features
Updating GroupActivity
Observing state changes on the GroupSession
GroupSessionMessenger API

[[Build custom experiences with Group Activities]]

# Picture in Picture
Allows oyur content to start playing instantly
No need for explicit user interaction to begin
Saves your users a step, less friction

[[Delivering intuitive media playback with AVKit - 19]]

1.  GroupSession deliveredin background
2.  GroupSessionw ill indicate this app should start in background
3.  Set up PIP
4.  Start coordinating playback and join session
5.  System starts content automatically

## What if your app requires user interaction?
* User might have to sign in
* Content might not be available
* GroupSession provides APIs for foreground


# Playback coordinator

Coordination with AVPlayer
Asset selection
Suspending coordination
Custom player implementations

## AVPlaybackCoordinator
Shares playback across devices

AVPlayerPlaybackCoordinator => tied to a particlar player, recommended
AVDelegatingPlaybackCoordinator => works with other playback API

Player asks the coordinator if it can play.  UI reperents this with a waiting spinner.  Coordinator will then send the command to the remote device, it also enters a waiting state.  Coordinators give everyone time to begin playback.  all devices begin playback together.

Seek API is intercepted and the coordinator forces the playback to wait while it shares the command.  Everyone gets time to complete the seek, playback resumes.

Coordinator only applies state to the other player when they're playing *identicial content*.  For now, content is identical if created from the same URL.  

Per-item commands enable safe joining and item transitions

we always prefer the group state.  So group state overrides local state.  

This is a complicated example involving one user not having good network.  Basically, I can transition my device to a new item, but the user with bad network cannot.  So I play ahead, and send commands to the other user, which are not applied because they refer to a different item.  But when they transition to the new item, the commands are applied, and he fastforwards to keep up with me.

Order matters.  

1.  seekToTime
2.  play
3.  replacecurrentItemWith

In this order, our initial configuration cannot affect anyone else.  In this order, the playback coordinator can decide if we are first and our state should be shared, or if another state already exists and should override our state.

Audit transport commands, should this affect everyone?  Usually if it affects playback UI.  So call API as usual.

What to do with other calls not ocming from playback UI?  

Usually these are automatic pauses because your app has encountered some system event.  Consider not pausing at all.  users may prefer to stick with the group even if it has some drawback.

If you have no other choice, you have 2 options discussed later.

## Identity
Two assets are the same if created from URL.

Problems:
1.  Content may be downloaded vs streamed

Implement a custom identifier.  Coordinator will ask its delegate for an identifier when enqueueing an item in the player.  Important that times match.  e.g. content that is automatically injected may be a problem.  Devices may get out of sync.  The right way to approach this problem is to move ads and intersitials into a separate player.  

[[Explore dynamic pre-rolls and mid-rolls in HLS]]

## Know your assets
* Use cusotm identifiers to match the same content behind different URLs
* Prevent content from getting out of sync
* Manage personalized interstitials with AVPlayerIntersititialEvent
* Use `PROGRAM-DATE-TIME` flag

## Suspending coordination
e.g. during an alarm on one device.  

`AVCoordinatedPlaybackSuspension`.  Represents a participant who si temporarily separated from the group.  This participant doesn't affect the group and isn't affected by the group.

## Automatic suspensions
AVPlayerPlaybackCoordinator adds suspensions for automatic pauses.
For example, interruptions, stalls, interstitials
Automatic suspensions end when the player resumes playback
Ending the suspension will rejoin the group

## Manual suspensions
Try to keep users together as much as possible.  But let's say we don't want to.

```swift
class AVPlaybackCoordinator {
    func beginSuspension(for reason: AVCoordinatedPlaybackSuspension.Reason) -> AVCoordinatedPlaybackSuspension
}

class AVCoordinatedPlaybackSuspension { 	
    func end()
    func end(proposingNewTime newTime: CMTime)
}
```

End the suspension to rejoin the group's current time and rate
You can optionally propose a new time ot the group.  

# Recap
When are commands coordinated?
* AVPlaers connected through AVPlayerPlaybackCoordinators
* AVPlayers' current AVPlayerItems have same URL or same identifier
* No AVCoordinatedPlaybackSuspension

Other APIs:
Other participants to wait?  `.suspensionReasonsThatTriggerWaiting`.  
`.otherParticipants`
`suspensionReasons` 

`.waitingForCoordinatedPlayback`
to override waiting, `.playImmediately(atRate:)`.
Note this can cause other participants to miss content.

`.rateDidChangeNotification` has the originating participant.

* Low quality zero latency => Not supported; deprecated
* Time domain => Supported, new default
* Varispeed => supported
* Spectral => supported

Do nto use `AVPlayer.setRate(time:atHostTime:)` with a coordinator.  It's vital that the coordinator take charge of timing.

# Custom player implementations
Most concepts we discussed apply to the delegating playback coordinator.  But you have to implement teh commands.  

1.  Your UI tells the coordinator
2.  Coordinator forwards to your player
3.  Tell coordinator when you transition to a new item

It is your repsonsibility to maintain the requested timing.  Be careful with things that play/pause the player.  If you cannot keep up, you should communicate to the coordinator using a suspension.
You must manage all suspensions for system events, there are no automatic suspensions.  Use system-provided reasons where appropriate.
Connecting to an AVPlayerPlaybackCoordinator on the other end must use custom identifiers.  



