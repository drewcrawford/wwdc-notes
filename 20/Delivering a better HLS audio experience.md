#hls 
Improve data efficiency and fidelity of your audio streams?

We're going to discover how to deliver a better HLS audio experience.

I recommend you read this first:

https://developer.apple.com/documentation/http_live_streaming/hls_authoring_specification_for_apple_devices

# New audio codecs
* xHE-AAC.  This audio codec is very efficient at low to medium bitrates.  Below say 200kbps.
* FLAC and Apple Lossless.  

## xHE-AAC
a.k.a. MPEG-D USAC.  Specifically tuned to speech reproduction.
Also good at a general purpose audio codec.
* significantly better results at low dat rates
* more efficient at medium data rates

## relationship to AAC family
* AAC-LC.  `CODECS=mp4a.40.2`.  Recommended at rates as low as 96kbps.
* This evolved into another (containing) codec, HE-AAC.  This is `CODECS=mp4a.40.5`.  This adds SBR.  High frequences are reconstructed from lower frequencies in the core AAC encoding.  We recommend this as low as 48kbps.
* HE-AAC v2 with another coding tool called "parametric stereo".  This reconstructs a second audio channel from a mono channel with some additional audio data.  We recommend this as low as 32kbps.

All 3 codecs have a leve of interoperability.    You can decode an HE-AACv2 with with an HE-AAC decoder.  However, you'll only get 1 cannel of audio because the earlier codecs don't know how to deal with parametric stereo.

## xHE-AAC
Backwards compatibility isn't there.  We refined the tools.  Important to identify xHE-AAC correctly. `CODECS=mp4a.40.42`
This is such an advanced codec that we recommend its use down to 24kbps.

## Loudness and Dynamic Range Control
Existing HLS authoring guidelines for all audio encodings
* loudness metadata
* DRC metadata

Extra metadata that allows the media system to adjust the audio signal levels to reduce level differences.

These are mandatory for XHE-AAC in CMAF ("shall")
* remains optional for AAC in CMAF ("should")

DRC usage with HLS will become increasingly important.

## HLS support for xHE-AAC
* Use `CODEC="mp4a.40.42"`
* AVPlayer supports mono and stereo config.
* No multichannel support at this time.
* fmp4 only
* Common encryption only

## anticipated xHE-AAC use cases
* recommended bitrates for stereo: 24-160kbps
* Use scenarios
	* additional low-bitrate audio variants
	* Reach customers on datarate constricted devices.  e.x., apple watch

[[What's new in streaming audio for apple watch]]

## additional use cases
* Parallel some, or all of the AAC family with xHE-AAC
	* Provide high-fidelity variants with the same bit budget
* Opportunity to add DRC support to your playlists

How to coerce automatic selection?  `SCORE` attribute.  This is because an xHE-AAC might be better than some other variant that's at a slightly higher bitrate

[[Improve stream authoring with HLS tools]]

## lossless audio
* `CODEC="fLaC"` or `CODEC="alac"`
* AVPlayer supports all specified channel configurations
* fmp4 only
* common encryption only

## anticipated lossless use cases

* Additional high-bitrate audio variants
	* when bandwidth is plentiful
	* When audio fidelity is high fidelity

# Multichannel use cases
* both FLAC and Apple Lossless support 5.1 and 7.1
* Be aware of the supported channel layouts and channel orderings
	* Apple lossless follows MPEG (C,L,R...)
	* FLAC follows SMPTE/ITU-R (L,R,C,...)

## data rates
Lossless audio is nearly 4x more at 16 / 48khz
and 24/96khz is nearly 3mbit.
These are merely average datarates.
Note that peak datarates are even higher.

Multichannel is even higher.  Various charts about how many megabits the audio is, but it's like 1-10mbit.

* add multichannel loslsess if this has appeal for your content or service
* If you do, also include multichannel AAC
* compressor can encode it
* Note: multichannel AAC cannot be decoded to multichannel output everywhere
	* In these cases you might get stereo AAC
	* Note you still need to provide a dedicated stereo AAC track

Note that AAC is important because we will be forced to use multichannel in a multichannel config.  So if you don't provide AAC, we will be forced to use flac multichannel for example, which may stall due to poor network.

# wrap up
* xHE-AAC, FLAC< Apple Lossless
	* inclusion of DRC
* Multichannel audio considerations
	* Utilize multichannel AAC-LC, etc., in order to scale up

# next steps
Employ xHE-AAC to target low throughput
Or beset utilize available throughput
Use lossless codecs
If applicable for your service
