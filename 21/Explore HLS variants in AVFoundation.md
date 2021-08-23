#avfoundation 

# HLS variant inspection
```
#EXTM3U
#EXT-X-VERSION:7
#EXT-X-INDEPENDENT-SEGMENTS

#EXT-X-MEDIA:TYPE=AUDIO,NAME="English",GROUP-ID="stereo",LANGUAGE="en",DEFAULT=YES, AUTOSELECT=YES,CHANNELS="2",URI="en_stereo.m3u8"
#EXT-X-MEDIA:TYPE=AUDIO,NAME="French",GROUP-ID="stereo",LANGUAGE="fr",DEFAULT=NO, AUTOSELECT=YES,CHANNELS="2",URI="fr_stereo.m3u8"

#EXT-X-MEDIA:TYPE=AUDIO,NAME="English",GROUP-ID="atmos",LANGUAGE="en",DEFAULT=YES, AUTOSELECT=YES,CHANNELS="16/JOC",URI="en_atmos.m3u8"
#EXT-X-MEDIA:TYPE=AUDIO,NAME="French",GROUP-ID="atmos",LANGUAGE="fr",DEFAULT=NO, AUTOSELECT=YES,CHANNELS="16/JOC",URI="fr_atmos.m3u8"

#EXT-X-STREAM-INF:BANDWIDTH=14516883,VIDEO-RANGE=SDR,CODECS="avc1.64001f,mp4a.40.5", AUDIO="stereo",FRAME-RATE=23.976,RESOLUTION=1920x1080
sdr_variant.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=34516883,VIDEO-RANGE=PQ,CODECS="dvh1.05.06,ec-3", AUDIO="atmos",FRAME-RATE=23.976,RESOLUTION=3840x1920
dovi_variant.m3u8
```

Here we have 2 variants.  One is SDR, the other is dolby vision with atmos audio.

Maybe you badge content in your app.  Now in IOS 15 you can inspect the playlist to infer these.

1.  AVURLSAsset => master.m3u8
2.  Obtain variants

Some, like media bitrate are direct.  Others are subclasses.  `VideoAttributes`, `AudioAttributes`.  `RenditionSpecificVariants`


# HLS variants with downloads
* Introduced in 2016
* Reuse existing media library
* Supports FairPlay Streaming content protection

[[Discover How to Download and Play HLS Offline]]

In iOS 15, we're taking HLS download API and making more powerful

## Variant selection
* Influence via AVAssetDownloadTask options
* HDR
* Lossless audio
* Media bitrate
* Presentation size

## Variant qualifier
* Allows you to specify your variant preferences
* Can be constructed using NSPredicates

```swift
let peakBitRateCap = NSPredicate(format: "peakBitRate < 5000000")

let peakBitRateCapQualifier = AVAssetVariantQualifier(predicate: peakBitRateCap)
```

* Supports compound predicates
* Predicates can be created against any property
* Custom predicate constructors

## Content configuration
Represents a set of video, audio, and subtitle renditions

```swift
let variantPref = AVAssetVariantQualifier(predicate: NSPredicate(format: "videoAttributes.videoRange == %@ && peakBitRate < 5000000", argumentArray: [AVVideoRange.pq]))

let myMediaSelections : [AVMediaSelection] = [enAudioMS, frAudioMS, enLegibleMS] //English, French audio and English subtitle renditions 

let contentConfig = AVAssetDownloadContentConfiguration()

contentConfig.variantQualifiers = [variantPref]

contentConfig.mediaSelections = myMediaSelections
```

## Download configuration
* Created with `AVURLAsset`
* needs asset name and optionally an image (displayed in settings)
* Accepts content configurations

Primary, auxilliary

Specifying as auxilliary instructs AVFoundation to optimize and avoid downloading multiple video renditions

```swift
let asset = AVURLAsset(url: URL(string: "http://example.com/master.m3u8")!)
let dwConfig = AVAssetDownloadConfiguration(asset: asset, title: "my-title")

/* Primary content config */
let varPref = NSPredicate(format: "videoAttributes.videoRange == %@ && peakBitRate < 5000000", argumentArray: [AVVideoRange.pq])
let varQf = AVAssetVariantQualifier(predicate: varPref)

dwConfig.primaryContentConfiguration.variantQualifiers = [varQf]
dwConfig.primaryContentConfiguration.mediaSelections = [enAudioMS, frAudioMS, enLegibleMS] //English, French audio and English subtitle renditions 

/* Aux content config */
let auxVarPref = NSPredicate(format: "%d IN audioAttributes.formatIDs", argumentArray: [kAudioFormatAppleLossless])
let auxVarQf = AVAssetVariantQualifier(predicate: auxVarPref)

let auxContentConfig = AVAssetDownloadContentConfiguration()
auxContentConfig.variantQualifiers = [auxVarQf]
auxContentConfig.mediaSelections = [enAudioMS] //english audio
dwConfig.auxiliaryContentConfigurations = [auxContentConfig]

dwConfig.optimizesAuxiliaryContentConfigurations = true
```

Optimize your content configurations to `true`.  Allows AVFoundation to choose lossless variants such that..  same as content configuration.

False may cause lossless variant to be downloaded indepedently.  

```swift
let myAssetDownloadDelegate = MyDownloadDelegate()
let avurlsession = AVAssetDownloadURLSession(configuration: URLSessionConfiguration.background(withIdentifier: "my-background-session"), assetDownloadDelegate: myAssetDownloadDelegate, delegateQueue: OperationQueue.main)

let asset = AVURLAsset(url: URL(string: "http://example.com/master.m3u8")!)
let dwConfig = AVAssetDownloadConfiguration(asset: asset, title: “my-title”)

...

let downloadTask = avurlsession.makeAssetDownloadTask(downloadConfiguration: dwConfig)

downloadTask.resume()

let progress = downloadTask.progress
```

Can choose variants directly

```swift
/* Example for direct variant selection */

let asset = AVURLAsset(url: URL(string: "http://example.com/master.m3u8")!)
let dwConfig = AVAssetDownloadConfiguration(asset: asset, title: "my-title")

/* Primary content config */
let myVariant : AVAssetVariant = ...
let myMediaSelections : [AVMediaSelection] = ...

let variantQf = AVAssetVariantQualifier(variant: myVariant)

dwConfig.primaryContentConfiguration.variantQualifiers = [variantQf]
dwConfig.primaryContentConfiguration.mediaSelections = myMediaSelections

/* Aux content config */
let myAuxVariant : AVAssetVariant = ...
let myAuxMediaSelections : [AVMediaSelection] = ...

let auxVariantQf = AVAssetVariantQualifier(variant: myAuxVariant)

let auxContentConfig = AVAssetDownloadContentConfiguration()
auxContentConfig.variantQualifiers = [auxVariantQf]
auxContentConfig.mediaSelections = myAuxMediaSelections
dwConfig.auxiliaryContentConfigurations = [auxContentConfig]
```

# Wrap up
* Inspection of HLS variants
* HLS variants with downloads
	* Variant qualifier
	* Content configuration
	* Download configuration
	* NSProgress support

https://developer.apple.com/streaming/


