#tvOS 

# tvOS Picture-in-picture
Can have multiple videos play simultaneously.
Button to swap.

# implementing tvOS picture-in-picture
## project setup
Note that these are the same for e.g. iOS
* need to add PIP capability.  It's inside "Background Modes".  "Audio, airplay and picture-in-picture"
* Configure AVAudioSession


```swift
let audioSession = AVAudioSession.sharedInstance()
do {
    try audioSession.setCategory(.playback)
} catch {
    print("Setting category to AVAudioSessionCategoryPlayback failed.")
}
```


# Working with standard playback UI
* PiP enabled by default
* Pip lifecycle `AVPlayerViewControllerDelegate` methods in tvOS 14
	* restore.  When implementing this method, fast restoration is ideal, and avoidng animations is a good idea.  If your app takes too long, it will be terminated.

# Custom playback UI
`AVPictureInPictureController`.  Your app will use this to control the pip.
New API to manage custom UI.

If `canStopPictureInPicture` is false, it indicates that your app should display a "start pip" affordance.  That affordance should call `startPictureInPicture()`.  Conversely, when it's true, you want to display affordances for `swap` and `stop`.  
No special API for swap.  Your app should call `startPictureInPicture()` when starting a swap, and the system will take care of the rest.

To stop, call `stopPictureInPicture()`.  Only call this when the user has selected your stop affordance.

`canStopPictureInPicture` can change at any time.  So your UI must be up to date, observe this property

```swift
_ = pipController.observe(\.canStopPictureInPicture) { controller, change in
    // Update your UI
    if controller.canStopPictureInPicture {
        pipActions = [.swap, .stop]
    } else {
        pipActions = [.start] //display start affordance
    }
}
```

Please read the HIG which goes into details about how to display these affordances

# Publishing Now Playing state
`MPNowPlayingSession` ties `AVPlayers` to a session
In the pip case, you have a nowplayginsession for pip, and non-pip.
Players used in Picture in Picture must be tied to `MPNowPlayingSession`

```swift
final class CustomPlayerViewController: UIViewController {

    init(player: AVPlayer) {       
        let playerLayer = AVPlayerLayer(player: player)       
        pictureInPictureController = AVPictureInPictureController(playerLayer: playerLayer)

        nowPlayingSession = MPNowPlayingSession(players: [player])
    }

    private func publishNowPlayingMetadata() {
        nowPlayingSession.nowPlayingInfoCenter.nowPlayingInfo = // Your Now Playing info
        nowPlayingSession.becomeActiveIfPossible()
    }
}
```

Migrate away from `MPNowPlayingInfoCenter.default()`, system ignores these in pip situation.
System decides which NP information to display
Always publish NPI.

`AVPlayerViewControllerDelegate` methods
1.  `playerViewControllerWillStartPictureInPicture`
2.  `restoreUserInterfaceForPictureInPictureStopWithCompletionHandler`
3.  `playerViewControllerWillStopPictureInPicture`
4.  `playerViewControllerDidStartPictureInPicture`
5.  `playerViewControllerDidStopPictureInPicture`

Swapping across applications.  Imagine your app is providing full-screen movie, and another app is doing the pip.  When we swap, your content is going *into* pip.

1.  `playerViewControllerWillStartPictureInPicture` (swap initiated, animations have not started yet)
2.  `playerViewControllerDidStartPictureInPicture` (swap completed, animations have completed)
3.  your app is backgrounded.  As part of the swap, another app became the foreground.  This is the same sequence you would see on #ipados when the user returns to the homescreen.


Now let's consider the case that *you* are pipped, and the other video is fullscreen.  During a swap, your pip content will take over the screen.

1.  your app is foregrounded
2.  `playerVIewController(_, restoreUserInterfaceForPictureInPictureStopWithCompletionHandler`
3.  `playerVIewControllerWillStopPictureInPicture` swap initiated, animations have not started
4.  `playerViewControllerDidStopPictureInPicture` swap completed, animations completed

# Interactive overlays
The swipe-up gesture is reserved.  Use `customOverlayViewController` for your interactive controls.

Your custom swipe up gesture recognizer wll stop working.
Users can access by swiping up or clicking button.  For an in-depth look at custom overlays, see last year's talk.

# Now playing info
`AVPlayerViewController` publishes Now Playing Info.  If you need to augment, use `externalMetadata` rather than talking to the framework directly.

# Architectural concerns
The delegate must exist during PiP
Users can navigate throughout your application
Your delegate must not be part of your view hierarchy.  It should probably be a separate object that can persist.

Full-screen and Pip videos trade places
Dismiss one VC and present the other
Embedding `AVPlayerViewControlelr` in another VC poses additional challenges.  Do you want to recreate the parent when a pip video is restored?  Consider the navigational hierarchy.  What should the user see when they dismiss the video?

If you have a custom playback UI, you may be pausing videos when the app is deactivated.  But don't do this in PiP.

# demo

# summary
* Use `AVPlayerViewController`
* If you need a custom UI, follow the HIG
* Do not install a swipe-up GR for your own overlaid UI
* Adjust application architecture

