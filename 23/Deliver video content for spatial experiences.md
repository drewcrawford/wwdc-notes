Learn how to prepare and deliver video content for visionOS using HTTP Live Streaming (HLS). Discover the current HLS delivery process for media and explore how you can expand your delivery pipeline to support 3D content. Get up to speed with tips and techniques for spatial media streaming and adapting your existing caption production workflows for 3D. And find out how to share audio tracks across video variants and add spatial audio to make your video content more immersive.

# Deliver 2D content

encoding video, audio, cpations
HLS packaging

fMP4 timed metadata.  Please note, http live streaming page on apple developer website.

Delivery of 2D audiovisual content is the same
Media fucntionality is built upon existing technologies
AVFoundation, HLS, standards-baed fmp4, webvtt, etc.
extended for new experience pardigm

See [[Create a great spatial experience for video playback]]

source video, etc.  Make choices about how to use HEVC, support for existing 2d av media, etc.

* up to 4k resolution video playback
* 90hz display refresh rate
* 96hz may be used automatically
* color piepline supports standard and hdr content
encode audio

may want to deliver spatial audio, along with stereo audio track, etc.  Ensures a great experience for spatial audio, etc.

Prepare captions.  Captions include subtitles and closed captions to cover different languages and roles.  Subtitles is used for spoken text, etc.

closed captions -> audio can't be heard.  transcribe not only dialog but also sfx, etc.

produce caption files and audio formats, ex vtt.  

## package for HLS delivery
translate source media into segments.  Apple's HLS tools.  

fragmented mp4 -> start with encoded video file and generate a number of resources, m4s.  These segments are retrieved by client devices.

subtitle files -> also require segmented.  `mediasubtitlesegmenter`.  

Collection of HLS resources is hosted on the webserver for http delivery.  One server can serve clients directly, or origin server can serve network. These resources are delivered to client devices for playback.
# Deliver 3D content

A track or stream of frames.
3d video = stereoscopic video
a left eye and right eye image for every frame.

All video frames should be carried in one video track.
Both left and right eye images for a time are in each compressed video frame.
Coding efficiency
Support on apple silicon.
Compatible with 2D video.

Introduce multivew HEVC.  Also known as MV-HEVC.
Extensions to HEVC.
carries multiple views in each compresse frame
pair of LR images.

MV_HEVC is HEVC, so apple silicon used
carries base HEVC 2D view data as-is
uses different between left and right views
stores original plus this difference (2d plus delta).
Efficiency between 2d frames and between views with 3d frames.

vexu (video extended usage).  Lightweight, discoverable signal that the video is stereoscopic.  

encoding.  Similar to 2d video production using hevc or other codecs
3d video requires multivew hevc to carry stereoscopic frames
local movies contianing mv-hevc should behave like 2d video.

screen plane - no parallax
negative parallex - in front
positive parallax - behind the screne plane.

depth conflict is created for thigns like subtitles if things collide.  How do we handle horizontal, vertical, dynamic type, etc.?

You can reuse your 2d captions by adding new timed metadata to the 3d video segments.

Important to avoid depth conflict in playback of 3d video.
Done by characterizing parallax across left and right views of the video frame.
This is termed a *parallax contour*.  
metadata item in timed track.

Describes a 2d tiling with minimum parallax value.  Each video frame's presentation should be associated with a metadata.  10x10 is a good size.

Start with left/right views.  Build parallax maps.  Package into metadata payload.Carry in metadata samples, write into timed metadata track.  Associated with corresponding video.

Multiplex into same movie, so taht HLS Packaging produces both video and metadata.

Reuse your 2d captions with your 3d video
your 2d proudction remains the same.
use same audio as 2d
spatial audio playback with head tracking
share same audio across 2d and 3d experiences.

Share same audio across 2d/3d experiences.  Updated HLS tools take care of details with 3d cassets makign process identical to 2d.

Most production systems will be able to use new specs being released.

If building your own playlist, take note of a few changes.
New REQ-vide-layout attribute
For EXT-X-STREAM-INF tag for video content
required on 3d streams to indicate stereoscopic video
video channel specifier can have CH-STEREO or CH-MONO or a mix
ext-X-VERSION is updated to 12. (hls verison 12)


3d packaging and delivery
* prepare your source assets
* Use updated packaging tools
* hosting for delivery is the same
* consider comfort
	* 3d visual experiences should be comfortable to watch
	* potential comfort issues: extreme parallax, high motion, window violations
	* screen size or FOV may affect comfort

| media type     | 2d content                               | 3d content        |
| -------------- | ---------------------------------------- | ----------------- |
| video          | use a 2d codec (hevc)                    | use mv-hevc video |
| audio          | use preferred audio codecs               | same as 2d        |
| cpations       | use standard subtitle format (ex webvtt) | same              |
| timed metadata | n?A                                      | Include parallax timed metadata in MV-HEVC Video segments                  |

# Wrap up
* bring your 2d assets to 3d
* use mv-hevc for 3d videos
* include parallax metadata for captions in 3d

[[Create a great spatial experience for video playback]]

# Resources
* https://developer.apple.com/av-foundation/HEVC-Stereo-Video-Profile.pdf
* https://developer.apple.com/av-foundation/Stereo-Video-ISOBMFF-Extensions.pdf
* https://developer.apple.com/av-foundation/Video-Contour-Map-Metadata.pdf
