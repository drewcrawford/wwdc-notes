# Improve stream authoring with HLS tools
#hls

# Changes to HLS tools
Last year, low-latency HLS tools were in a separate package.  Since it’s coming out of beta, we’re using a single tools distribution.

Media stream validation has a new “immediate” option.
Hlsreport is now a compiled binary rather than a script.
New Audi codecs

[[Delivering a better HLS audio experience]]

# Using the Low-Latency HLS tools
There are several talks this year
[[Low-Latency HLS Updates]]
[[Introducing Low-Latency HLS - 19]]

* Tsrecompressor
	* Generates video & audio for media stream segmented

* Mediastreamsegmentor can produce `EXT-X-PART` tags and partial segments
* Implementing the Delivery Directives interface
	* `ll-HLS-origin-example.go`
	* `lowLatencyHLS.php`

## Creating a simple LL-HLS stream
1.  Transport stream via UDP
2. Media stream segmenter, produces both
	1. Media playlist
	2. And media segments

Media playlist goes to interface script, which returns a modified playlist.

Master playlist.  

```
tsrecompressor
—generate-input
—hardware
—preview-port ip:port
—low-quality-port ip:port
—output-port ip:port
```
Note that each output port needs its own mediastreamsegmentor
```
mediastreamsegmenter 
—part-target-duration-ms 1002
—target-duration 4
—sliding-window-entries 16
—delete-files
—date-time
—file-base /Path
ip:port
```

Go script
* Create server
* Serves synthesized playlist when asked for m3u8
Php script
* Uses existing server
* Multiple copies of php script, one per variant
Both require variants to be in subdirectories of one directory


# Using the new audio codecs in master playlists
* Unified speech and audio codec.  MPEG-D USAC (Extended HE-AAC) (xHE-AAC)
* CODECS=“mp4a.40.42”
Apple lossless, `CODECS=“alac”`

Free lossless audio codec (FLAC)
CODECS=“fLaC”

## How to build up a master playlist
Choose your video resolutions and codecs
Choose your audio languages
Choose audio codecs and number of channels
Make one audio group per audio codec/channel count pair
Replicate variants for each audio group

## Make audio groups
Always have a stereo AAC group
If you have multichannel lossless then also have multichannel AAC
We don’t want to switch the number of channels.
We don’t switch codecs except
* Between AAC codecs
* Between lossless and AAC

## Replicate variants per audio group
Groups with switchable codecs can spread across variants
* Associate low audio bit rates with lower video bit rates

Groups with distinct codecs should copy every variant
* This only creates more entries in the master playlist, not copies on disk

## SCORE attribute
Attribute on `EXT-X-STREAM-INF` and `EXT-X-IFRAME-STREAM-INF` tags
* SCORE = <decimal floating point>
* Support is in iOS 13, macOS 10.15, tvOS 13
* Used to influence choice of variant
* Should be on every variant, otherwise it is ignored
* After we have filtered variants based on playability and network bandwidth, highest score wins

# Wrap up
We’ve made changes to the tools
Audio groups
