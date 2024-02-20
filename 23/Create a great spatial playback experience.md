Get ready to support video in your visionOS app! Take a tour of the frameworks and APIs that power video playback and learn how you can update your app to play 3D content. We'll also share tips for customizing playback to create a more immersive watching experience.

Optimized for the platform, takes advantage of media capabilities, integrates with system.

# Media experience
If you've used media apis, this may feel familiar.  This platform builds on same APIs and extends them for unique capabilities.

AVFoudnation handles all the work of playing movies.
AVKit builds on AVFoundation as UI frameworks to create a playback experience for each platform.


[[Deliver video content for spatial experiences]]

Render performantly using realitykit.  So that video can be composited seamlessly.  Audio also responds, etc.

AVPlayerViewController has been extended to make use of the power of realitykit. Refined experience.  Includes playback controls, capabilities, etc.

* Build with the matching SDK
	* compatible iOS apps will get an iOS-compatible experience.
* AVPlayerViewController
* Fill the window

Wrap the code in a UIViewController representable.  

AVPlayerVC and AVPlayer do a lot of the heavy lifting so you don't have to.

While playing, do nothing and they will disappear on their own.  Look at the screen and tap, etc.

Grab the corner of the screen to resize.  Notice that as the screen resizes, it animates smoothly to match the aspect ratio of the video.  Adjust volume by turning digital crown, or use digital crown to open environment.

Playback controls to see features.  Player interface.  Top right is the volume control.  Volume can also be adjusted via digital crown.  Play/pause, back/forward buttons.  In bottom middle is the scrubber.  Bottom right is this button with more options.

Sometimes a video isn't my only focus.  Turn off dimming effect to see what's around you.
# Advanced features

Thumbnail scrubbing and Trick Play.
Controls will automatically display a thumbnail when scrubbing for HLS streams.  I-frame only playlist or Trick Play
Width of 145px
See HLS spec section 6, "Trick Play"

Logo, recap, ad.  Interstitials enable this ability.  Controls automatically reflect them with timeline indicator.

configure programmatically via AVPlayerInterstitialEventController.  [[What's new in HLS Interstitials]]

Additional UI options
* contextual actions
* Custom info view controllers
* Same API as other platforms
[[What's new in AVKit]]

Immersive spaces

Use your own 3D assets
Screen auto-docks
Detached controls

Build immersive spaces tailored to player's placement

[[Go beyond the window with SwiftUI]]

Thoroughly refined and designed for great platform experience.  Please send feedback.

There could still be rare situations where someone needs ot build custom playback controls.  Sned us feedback, we recommend using AVPlayerVC.  Hide controls and add an overlay.  preferred over a lower-level API because AVPlayerViewController provides many system integration features beyond just playback controls

we intend to make updates
# Other use cases

Play video in a window
In a document
As a preview alongside other content
AVPlayerViewController presented inline
Used whenever you're not covering the window
Redesigned controls
AVPlayerLayer composites with other content (so it can't display 3d video?)

RealityKit entity video
splash screen, video transition
no playback controls
no system integration
For those, use VideoPlayerComponent.

This connects an RK entity to AVPlayer.  
Aspect ratio correct mesh
Captions
Prefer over AVPlayerLayer
RK's optimizations provides better performance than layer
and supports 3d video format

[[enhance your spatial computing app with RealityKit]]

May want to use video in a 3d scene as an effect.
* video as a texture on geometry
* RealityKit VideoMaterial to display on arbitrary geometry
* Not aspect ratio correct
* No captions

[[what's new in realitykit]]
# Conclusion

Notice the use of inline player with custom UI controls for preview.  Then, play the movie.  The player UI appears an dimmediately docks into the immersive space at a fixed size and location for optimal viewing.

When screen is docked, playback controls detach and come closer to make them more convenient.  

A contextual action button labeled 'play next' appears in bottom right corner.

We've covered spatial experience.  Large screen, etc.  It's nice.

Several ways to display video.

|  | Captions support<br>maintain aspect ratio | realityKit rendering | playback controls | full spatial platform experience |
| ---- | ---- | ---- | ---- | ---- |
| AVPlayerViewController FS |  yes | yes | yes | yes |
| AVPlayerViewController inline | yes |  | yes |  |
| AVPlayerLayer | yes |  |  |  |
| VideoPlayerComponent | yes | yes |  |  |
| VideoMaterial |  | yes |  |  |
|  |  |  |  |  |



# Wrap up
Sample project: DestinationVideo
[[What's new in AVKit]]
[[enhance your spatial computing app with RealityKit]]
[[Deliver video content for spatial experiences]]

