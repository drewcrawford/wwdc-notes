#avfoundation 

1.  Source material
2.  Media encoder
3.  Segmenter
4.  Web server

# AVAssetWriter
Outputs media data in fragmented mp4 format (for HLS)
Not for mepeg-2 transport stream, ADTS, and mpeg audio

## Use cases
Video on demand material.
1.  Movie
2.  AVAssetReader
3.  AVAssetWriter
4.  header, segment1, segment2, etc.

May also AVCapture for live data instead.
# Fragmented mp4
Based on ISO base media format
Supported by HLS since 2016

## Format structure
Regular mp4 has filetype, movie box (audio/video codecs, location data, etc), and media data.  Note that other boxes can be in most orders.

Fragmented movie has file type, then random interleaved movie box and media data.  The extra movie boxes are called "movie fragment".  

fmp4 -> file type, movie, movie fragment, media data.  Then more media fragment, more media data, etc.  This is sort of a "defragmented" fragmented movie.  Note that here, movie box does not contain reference to media data.  And order between movie fragment and movie data is reversed.

```swift
// Instantiate asset writer
let assetWriter = AVAssetWriter(contentType: UTType(AVFileType.mp4.rawValue)!)

// Add inputs
let videoInput = AVAssetWriterInput(mediaType: .video, outputSettings: compressionSettings)
			
assetWriter.add(videoInput)
```

## configure AVAssetWriter
```swift
assetWriter.outputFileTypeProfile = .mpeg4AppleHLS
assetWriter.preferredOutputSegmentInterval = CMTime(seconds: 6.0, preferredTimescale: 1) //outputs at this interval.  HLS requires integer intervals?
assetWriter.initialSegmentStartTime = myInitialSegmentStartTime
assetWriter.delegate = myDelegateObject
```

[[Working with Media in AV Foundation - 11]]

## delegate methods
```swift
//preferred
optional func assetWriter(_ writer: AVAssetWriter, didOutputSegmentData segmentData: Data, segmentType: AVAssetSegmentType)


optional func assetWriter(_ writer: AVAssetWriter, didOutputSegmentData segmentData: Data, segmentType: AVAssetSegmentType, segmentReport: AVAssetSegmentReport?)
```

```swift
public enum AVAssetSegmentType : Int {
    case initialization = 1  //file type, movie box
    case separable = 2 //movie fragment, media data, ...
}
```

You package the data into segments, and write segment files.  A pair of a file type box and a movie box are initialization segment (header.mp4).  The movie fragment, media data, are a segment (segment1.m4ss,segment2.m4s).

## playlist
AVAssetWriter does not write the playlist.  You write the playlist.  That is why the delegate delivers `AVAssetSegmentReport`.  You can create the playlist and the I-frame playlist based on the information in the segment report.  See our sample code.

https://developer.apple.com/streaming

## Segment interval
CMAF requires every segment to start with a sync sample.  Apple HLS also prefers this.  Not only to video, but also audio.

Therefore, for passthrough, only 1 input can be added.  

Can encode video samples.  If you specify output settings for compression, in `AVAssetWriterInput`, we encode.  

## Master playlist
Deivers video / audio as separate streams.  Streams at various bitrates or languages.

Need to create multiple instances of AVAssetWriter.

https://developer.apple.com/streaming

If you require different segmentation than preferredOutputSegmentinterval does, you can do it on your own.  e.x., you may want to output data, not after sync sample, but after some interval has been reached, but before the interval has been reached ?

To signal custom segmentation, set `preferredOutputSegmentInterval = .indefinite`.

```swift
// Set properties
assetWriter.outputFileTypeProfile = .mpeg4AppleHLS

assetWriter.preferredOutputSegmentInterval = .indefinite

assetWriter.delegate = myDelegateObject

// Passthrough
let videoInput = AVAssetWriterInput(mediaType: .video, outputSettings: nil)
```


You call `flushSegment` to output the data.  Appends all sampels added since the previous call.  Must call `flushSegment` prior to a sync sample, so the next fmedia can start with the sync sample.  Otehrwise, an error that indicates that a fragmented media data started with non-sync sample occurs.

In this mode, you can mux audio/video.  But if both audio and video has sample dependencies, it would make it difficult to align both media evenly.  In that case, you should consider packaging media/audio as separate streams.

## Audio has dependencies

```swift
extension AVAssetTrack {
       /* indicates whether this audio track has dependencies (e.g. kAudioFormatMPEGD_USAC) */
    open var hasAudioSampleDependencies: Bool { get }
}
```

# CMAF
ISO/IEC 23000-19
constrains fmp4
supported by apple hls

Some constraints are not required if you target hls, but consider if you need broader support.

# Audio priming
For reasons, AAC requires data before an audio sample is encoded.  Therefore encoders add silence before the first true audio sample.  This is called "priming".

Most common priming is 2112 samples, which is just about 48ms, assuming 44.1khz.

If audio/video start at the same time, audio will be delayd by the priming.  CMAF adopts an "edit list" in the audio track to compensate for thi sissue.  The priming is edited out.

Historically, apple HLS hasn't used the edit list.  So if you specify HLS profile, edit list is not used.  Instead, the `baseMediaEncoderTime` is shited backwards.

However, the `baseMediaDecodeTiem` cannot be earlier than 0 since it is defined as a uint.  One solution is to shift both the audio and video forward by the "same time offset".  This way, they can agree.  In HLS, playback starts at the earliest presentation of the video sample, regardless of what the coordinates say.

We recommend shifting time by adding a certain time to all samples.  

[[Validating HTTP Live Streams - 16]]

# Wrap up
AVAssetWriter now outputs fmp4 format
