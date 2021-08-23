# What is spatial audio
Headphones: in-head experience.  Speakers move with us.  Not a theater-like experience.

Spatial audio
* Theatre-like experience
* Psychoacoustic technology
* Multichannel
* Stereo
* Audiovisual and audio only

Multichannel audio
* In HLS variant streams
* In Tracks within regular media files
* Via W3C MSE through WebKit


# Media experience
Stereo upmix.
Make spatial audio more compelling to adopt and offer.

Multichannel is higher bitrate and that causes problems, right?  How do you fit both in a constrained environment?

Adaptive media experience.  When bandwidth is insufficient we change to upmixed stereo that still has spatial treatment.

* Best practices
	* Normalize volume levels
	* Provide DRC metadata

developer.apple.com/streaming

# API
four formats.  monoAndStereo, multichannel, combination.

Specify 0 to inhibit spatialization.


`allowedAudioSpatializationFormats` on `AVPlayerItem`.  

How to use APIs to discover if audio graph supports spatial audio?

If your app uses AVPlayer, these are managed for you.

`isSpatialAudioEnabled` -> port can render and customer permits it.  Observe notifications.

`AVAudioSession.spatialPlaybackCapabilitiesChanedNotification`.

`setSupportsMultichannelContent`.  Relay to customers that a spatial experience is available if conditions permit.


# Feature availability
In catalina, iOS 13, spatial audio is supported for builtin speakers.  2018 and later year-model devices.

In big sur, we add support for airpods pro/max.  For these, we allow 2016 and later devices.

Monterey, 15 releases.  We offer AVPlayeritem, video tag, AVSampleBufferAudioRenderer and limited webkit support.


# Demo
# Wrap up
* Offer multichannel audio
	* May not need to do anything, just by offering multichannel audio in your playlist
* Normalize audio to common loudness between variants
* Include DRC metadata
* Reach all customers on the last 3 releases

[[Explore HLS variants in AVFoundation]]

