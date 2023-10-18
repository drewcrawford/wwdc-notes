Discover how the Cinematic Camera API helps your app work with Cinematic mode videos captured in the Camera app. We'll share the fundamentals — including Decision layers — that make up Cinematic mode video, show you how to access and update Decisions in your app, and help you save and load those changes.

New cinematic API.  New API to build sample app which does playback/edit, realizing awesome capabiltiies of cinematic mode.

# What is cinematic mode?

We introduced in iPhone 13.  Tiny film crew in your pocket.  Brings you a camera with beautiful shallow DOF and natural focus follow.  

focus puller that anticipates keyframes, and creates smooth transitions.

cpatured right in the camera app on iPhone 13+.  Rendering preview as you record.  Accessible whether you are an aspiring filmmaker, or just like to add a magic touch to videos.

change aperture and thereby the amount of bokeh.  Redirect focus using alternative detections.  This shows post-capture editing in photos, but can also edit in iMovie, final cut pro, motion, etc.

With introduction of API, can use for playback and edit in your app.

Cinematic API - widely available on new OSes.  

# Dataflow
* rendered asset
	* exported, shared, played, etc.
* cinematic mode asset -> all the information needed to create the rendered asset.  source file, basically.

Let's change the narrative to be about the team player, etc.

To support NDE, cinematic mode asset has multiple tracks.
* video track, original quicktime movie as captured.  HDR, SDR, 1080p 30fps.  On iPhone 14 we also support 4k, 24/25/30.
	* can be played as a regular video.
* disparity
	* pixel shift between two cameras
	* map of relative disparity
	* lower resolution than video track
	* only used relative to other samples in the same map

[[Capturing depth in iPhone Photography - 17]]

metadata
1.  Rendering attributes: focus disparity - aperture f/number
2. focus shown as an overlay is decidexc by cinematic engine, while the aperture is chosen by the user.  Then the cinematic script iholds all the automatic scene detections.  This scene shows face, torso, etc.

focus decisions can be changed post-capture.  Chagne narriative, etc.

1.  input data
2. editing (nondestructive)
3. rendering
4. rendered asset
# Playback
PhotosPicker and filter for cinematic videos.  As a side note, if you don't already have assets, airdrop between devices.

use 'allp hotos data' in airdrop options.

fetch the underlying asset.
ensure request options to get original version.  

Play a rendered asset
* avplayer and avplayeritem
play/edit a cinematic mode asset
* add a custom video compositor

[[Edit and Playback HDR Video with AVFoundation]]

Cinematic API prefixed with CN
* setting up a rendering session (`CNRenderingSession`)
* provide metal command Q

quality can be set to different levels depending on your performance/quality constraints.

In the typical case, need multiple steps for multiple tracks.

get cinematic script from asset.  `CNScript` -> holds detections and focus decisions.  I'll go into more detail later.

Set out instruction and rendering section, composition, cinematic script, aperture, etc.  This custom instruction describes how to compose render to cinematic content.

* get buffers
* disparity, metadata, image, etc.
From metadata buffer I can get rendering frame attributes.  i can also change them.

With frame attributes I can make optional playback changes, ex by changing aperture fNumber.

# Editing

automatic decisions are decided on a range of paramters: who faces the camera, what's closer, what's interesting, etc.

first one is to add a weak user decision.  A weak decision only follows the track until the next base/user decision.  So it is overridden by future base decisions.

If we want focus to stick, we can add another weak decision, or we can choose a strong decision.  This keeps focus on a subject until the next user decision to focus elsewhere.

decision hierarchy works as follows.
1.  user decisions
2. base decisions fill the gaps

while both decisions revert to base decision when a track ends, a strong decision holds its focus as long as possible.

Iterate all detections. 

Enable new detection overlay to draw detections.  Face, head, torso, etc.  

Focus racking ahead of time.
Pass focus with smooth transitions according to update script.

demo

load changes
reload onto original script

additional features
* export
* object tracker
* custom tracking
* customized rendering changes

# Wrap up
* play cinematic mode assets
* edit by changing the Cinematic script
* save and load script changes


# Resources
* https://developer.apple.com/documentation/avfoundation
* https://developer.apple.com/documentation/cinematic
* https://developer.apple.com/av-foundation/
