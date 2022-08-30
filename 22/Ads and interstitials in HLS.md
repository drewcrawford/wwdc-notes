Lets them navigate and interact with content in real time, in sync.  However, some of you might have targeted ads or interstitials.  Makign it a challenge to coordinate playback.  Or some users receive ads and others don't.

# Coordinated playback
# Challenges
ads is tough for coordinated playback.  some users have ads and some don't.

Let's say that alice's geomegraphy requires statutory warnings.  And there might be ads within the program.  It oculd be of different durations.  Might pose a challenge when trying to play in sync.  For bob, banner might appear the beginning, but for alice, might appear after warnings.

# User experience
Consider a simpelr timeline where each has a single ad.  In the ideal case... we expect tiemlines to match.

Sometimes, they might get different durations, or someone might not have any ads.  We can either wait or not.

You can specify these policies at the coordinator by how you populate the array.

Default behavior: no waiting.  
* participant catches up with the group upon ad end
* Might miss content

Or, wait.
* Include `playingInterstitial` suspension in AVPlaybackCorodinator.suspensionReasonsthatTriggerWaiting
* controlSTatus = waitingToPlayATSpecifiedRate
* waitingReason = waitingForCoordinatedPlayback

# Stitched in ads
Specify sample accurate time ranges that represent ads/interstitials
```swift
class MyAVPlayerCoordinatorDelegate : NSObject, AVPlayerPlaybackCoordinatorDelegate
{   
    func playbackCoordinator(_ coordinator: AVPlayerPlaybackCoordinator,     interstitialTimeRangesFor playerItem: AVPlayerItem) -> [NSValue]
 {
        return interstitialTimeRanges
     }
}
```

we use this info to coordinate playback across the group.

Seeking into an interstitial range causes playback to snap to the beginning of that range.

This assumes that durations accurately reflect media durations of those segments.

For these to be considered SP compatable, we expect content durationt o match (exlcuding ads).  This is only for VoD, ad breaks must match for live.

# HLS interstitials
Alternatively you can use interstitials.  We introduced these to offer a different approach for scheduling ads.

[[Explore dynamic pre-rolls and mid-rolls in HLS]]

* ads are separate objects external to the content timeline
* Use EXT-X-DATE-RANGE tags to perform SSAI
* AVFoundation APIs for CSAI
* Automatic support for coordinated playback
* Specify waiting policy

When using HLS interstitials, we'd expect primary content durations to match.  


# Best practices
* MInimize wait/skip between participants
	* By providing ad breaks of similar durations
* For live content
	* Use HLS interstitials with undefined resume offset
	* Default policy - no waiting
* For VoD content
	* Wait - wiating policy
	* Use GroupSessionmessenger to share ad schedules
	* Show interesting content while waiting

# Wrap up
* set waiting policy
* AVF API for stitched in ads
* Automatic support for HLS interstitials
* Build an engaging SharePlay experience


[[Make a great shareplay experience]]



* https://developer.apple.com/forums/tags/wwdc2022-110380
* https://developer.apple.com/forums/create/question?&tag1=91030&tag2=468030

