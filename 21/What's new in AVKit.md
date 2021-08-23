#avkit

Engineer on the AVKit team.

# Picture in picture
Users can continue to enjoy their video content while multitasking with their device.

Watching a video fullscreen, can reply to the message while continuing to see content.  Video enters pip automatically.

For more info, see
[[Delivering intuitive media playback with AVKit - 19]]

If your video is playing inline, can allow it to automatically enter pip from homescreen

```swift
// New property on AVPlayerViewController / AVPictureInPictureController.
var canStartPictureInPictureAutomaticallyFromInline: Bool { get set }
```

Make sure to only set this flag to true when the playing content is intended to be the user's primary focus.

If not using AVPlayerViewControler, can use `AVPictureInPicturecontroller` to bring PIP into the picture.

```swift
func setupPictureInPicture() {
    // Ensure PiP is supported by current device.
    if AVPictureInPictureController.isPictureInPictureSupported() {
        // Create a new controller, passing the reference to the AVPlayerLayer.
        pictureInPictureController = AVPictureInPictureController(playerLayer: playerLayer)
        pictureInPictureController.delegate = self
        
        // Observe AVPictureInPictureController.isPictureInPicturePossible to update the PiP
        // button’s enabled state.
    } else {
        // PiP isn't supported by the current device. Disable the PiP button.
        pictureInPictureButton.isEnabled = false
    }
}
```

```swift
@IBAction func togglePictureInPictureMode(_ sender: UIButton) {
    if pictureInPictureController.isPictureInPictureActive {
        pictureInPictureController.stopPictureInPicture()
    } else {
        pictureInPictureController.startPictureInPicture()
    }
}
```

You might need a background mode?

We used to require AVController, now we have something for AVSampleBufferDisplayLayer.

 We have to rely on playback stat einformation provided via new AVPictureInPictureSampleBufferPlaybackDeleegate.  User commands forwarded here.
 
 1.  SetPlaying: user preseses play/pause
 2.  skipByInterval
 3.  timeRangeForPlayback => rendering timeline
	 1.  Use infinite duration to indicate live content
 4.  didTransitionToRenderSize => pip changes size, such as pinch to zoom. Take this into account when choosing media variants.
 5.  isPlaybackPaused => informs PIP UI whether to reflect paused or playing state.


# Mac full screen experience
In BS, video will fill entire window but not entire screen.  Now in macOS monterey, video will take up entire screen.  True fullscreen experience for native macOS and catalyst.

All catalyst will get this automatically (kinda sucks for watching WWDC sxs right now though)

Users can detach to true fullscreen in cases where it isn't fullscreen (full-window)

Lifecycle can be managed to provide a better UX.  

In order to achieve an optimal experience, need to make sure to keep player VC alive even if not in hierarchy.  Otherwise, full-screen playack will end.

1.  Keep a strong reference to playerViewController.  This way if the user navigates around in your app, it won't kill the FS.
2.  Once user exists FS, get willEndFullScreen callback.  Let go of reference here.

`restoreUserInterfaceForfullScreenExit` => opportunity to return to your app.

Returning `false` indicates restoration fails or is impossible.

# Wrap up
* PIP support for AVSampleBufferDisplayLayer
* Full screen improvements for Mac