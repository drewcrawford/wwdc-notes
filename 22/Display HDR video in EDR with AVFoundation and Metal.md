#metal 
[[Explore EDR on iOS]]
[[Display EDR content with Core Image, Metal, and SwiftUI]]

# HDR Video media in EDR apps
# Apple EDR video frameworks
Use the highest-level framework possible.

AVKit
* AVPlayerViewController
* transfer controls, chapter navigation, etc.
* play back HDR content as EDR
* If your application requires further processing of video frames, have to use something else.
AVFoundation
* Full-featured framework for time-based audio/digital media
* Play, create, and edit movies
* Play HLS streams
* AVPlayer, AVPlayerLayer
Core Video
* Pipeline model for digital video
* Easier to access and manipulate individual frames
* DisplayLink
* CVPixelBuffer
* CCVMetalTextureCache
Video toolbox
* Low-level framework
* direct access to hardware encoders and decoders
* conversion between raster image formats
* VTDecompressionSession => outside the scope of this talk
Core Media
* Defines the media pipelines.
* Low level datatypes and interfaces
* queues
## Usecases we will explore
* AVKit, AVFoundation
* AVPlayerLayer to playback HDR video to a specific layer
* Using AVPlayer and CADisplayLink
* Sending resulting CVP:ixelBuffers to CoreImage
* Using AVPlayer and CVMetalTextureCache
* Sending results to metal
# Using AVKit and AVFoundation
Play local or remote assets
Plays single media assets at a time
AVQueuePlayer => manage queuing and playing of sequential assets

For simple cases, AVPlayer + AVPlayerViewController
For your own views, AVPlayer + AVPlayerLayer.

First, how to use AVPlayerViewController

```swift
// Playing media using AVPlayerViewController
let player = AVPlayer(URL: videoURL)

// Creating a player view controller
var playerViewController = AVPlayerViewController()

playerViewController.player = player

self.presentViewController(playerViewController, animated: true) {
   playerViewController.player!.play()
}
```

Some applications need to play back HDR media into their own view
```swift
// Playing media using AVPlayer and AVPlayerLayer
let player = AVPlayer(URL: videoURL)

var playerLayer = AVPlayerLayer(player: player)

playerLayer.frame = self.view.bounds

self.view.layer.addSublayer(playerLayer)

player.play()
```


Those are the two straightforward workflows.  Many applications require more than simple media playback.

* real-time effects?
* color grading
* chroma keying
* image processing
* coreimage filters
* metal shaders

How to decode frames.  Access the frames from displaylink, send pixel buffers to coreimage, and render in metal.

```swift
// Opt into using EDR
let layer: CAMetalLayer
layer.wantsExtendedDynamicRangeContent = true

// Use half-float pixel format (some format that supports EDR)
layer.pixelFormat = MTLPixelFormatRGBA16Float

// Use extended linear display P3 color space
layer.colorspace = kCGColorSpaceExtendedLinearDisplayP3
```

[[Explore HDR rendering with EDR]]
[[Explore EDR on iOS]]

## Using AVPlayer
1.  Create AVPlayer from aVPlayerItem
2. Create an AVPlayerItemVideoOutput
3. Create and configure a CADisplayLink
4. Run DisplayLink to get pixel buffers

We will demonstrate CADisplayLink, use CV for macOS.

step 1: not shown

```swift
let videoColorProperties = [
    AVVideoColorPrimariesKey: AVVideoColorPrimaries_P3_D65, //most apple displays
    AVVideoTransferFunctionKey: AVVideoTransferFunction_Linear, //maintain range for eDR
    AVVideoYCbCrMatrixKey: AVVideoYCbCrMatrix_ITU_R_2020
]

let outputVideoSettings = [
    AVVideoAllowWideColorKey: true, //wide color
    AVVideoColorPropertiesKey: videoColorProperties, //above
    kCVPixelBufferPixelFormatTypeKey as String: NSNumber(value: kCVPixelFormatType_64RGBAHalf)
] as [String : Any]
   
// Create a player item video output
let videoPlayerItemOutput 
= AVPlayerItemVideoOutput(outputSettings: outputVideoSettings)
```
This not only decodes input into output file specified.  But also does color conversion.

Recall, a video can contain multiple clips in distinct color spaces.  AVfoundation manages this for us, this allows the resulting decoded video frames to be sent to metal that don't themselves provide color space conversion automatically.

```swift
// Create a display link
lazy var displayLink: CADisplayLink 
= CADisplayLink(target: self, 
                selector: #selector(displayLinkCopyPixelBuffers(link:)))

var statusObserver: NSKeyValueObservation?

statusObserver = videoPlayerItem.observe(\.status,
      options: [.new, .old],
      changeHandler: { playerItem, change in
        if playerItem.status == .readyToPlay {
          playerItem.add(videoPlayerItemOutput)
          displayLink.add(to: .main, forMode: .common)
          videoPlayer?.play()
        }
     })
}
```

Callback run on each display update.  Video displayer item observer => observe changes to properties.

Take a look at example implementation.

```swift
//might be called 60fps.
@objc func displayLinkCopyPixelBuffers(link: CADisplayLink) 
{
  let currentTime = videoPlayerItemOutput.itemTime(forHostTime: CACurrentMediaTime())
	
  if videoPlayerItemOutput.hasNewPixelBuffer(forItemTime: currentTime)
  {
      if let buffer 
      = videoPlayerItemOutput.copyPixelBuffer(forItemTime: currentTime, 
	                                          itemTimeForDisplay: nil) 
	  {
        let image = CIImage(cvPixelBuffer: buffer!)

        let filter = CIFilter.sepiaTone()
        filter.inputImage = image
        output = filter.outputImage ?? CIImage.empty()
    
        // use context to render to you CIRenderDestination
     }
 }
}
```

If no new pixel buffer, preceeding frame may remain onscreen for refresh.  If it succeeds, we have new frame.

Can send this to CoreImage for processing.
* CAn string one or more CIFilters
* GPU accelerated processing
* Not all CIFilters are EDR compatible
* `CIFilter.filterNames(inCategory: kCICategoryHighDynamicRange)`

Let's integrate Core Image

(this listing is same as previous?)

```swift
@objc func displayLinkCopyPixelBuffers(link: CADisplayLink) 
{
  let currentTime = videoPlayerItemOutput.itemTime(forHostTime: CACurrentMediaTime())
	
  if videoPlayerItemOutput.hasNewPixelBuffer(forItemTime: currentTime)
  {
      if let buffer 
      = videoPlayerItemOutput.copyPixelBuffer(forItemTime: currentTime, 
	                                          itemTimeForDisplay: nil) 
	  {
        let image = CIImage(cvPixelBuffer: buffer)

        let filter = CIFilter.sepiaTone()
        filter.inputImage = image
        output = filter.outputImage ?? CIImage.empty()
    
        // use context to render to your CIRenderDestination
     }
  }
}
```

Please refer to WWDC 20 talk "Applying CIEffects to video" (it doesn't exist?) for more info about this technique

# Using Core Video with Metal
Implementing this efficiently is a deep topic.  We recommend getting it from the cache for this talk.

* Get IOSurface from CVPixelBuffer
* Create MetalTextureDescriptor
* Create MetalTexture from the Metaldevice using `newTextureWithDescriptor`
Danger that textures may be re-used if not careuflly reference-counted
Not all PixelBuffer formats are natively supported by metal texture

because of these complications, we'll directly access from corevideo.

You can get a MTLTexture directly from the cache.
Manages CVPixelBuffer and MTLTexture bridge

MTLTexture to IOSurface mapping are kept alive
No need to manually track an IOSurface

1.  Get metal device
2. Create Core Video metal texture cache
3. Create a core video metal texture
4. Use and then release the core video metal texture

```swift
// Create a CVMetalTextureCacheRef

let mtlDevice = MTLCreateSystemDefaultDevice()

var mtlTextureCache: CVMetalTextureCache? = nil

CVMetalTextureCacheCreate(allocator: kCFAllocatorDefault, 
                          cacheAttributes: nil, 
                          metalDevice: mtlDevice, 
                          textureAttributes: nil, 
                          cacheOut: &mtlTextureCache)

// Create a CVMetalTextureRef using metalTextureCache and our pixelBuffer
let width  = CVPixelBufferGetWidth(pixelBuffer)
let height = CVPixelBufferGetHeight(pixelBuffer)

var cvTexture : CVMetalTexture? = nil

CVMetalTextureCacheCreateTextureFromImage(allocator: kCFAllocatorDefault, 
                                          textureCache: mtlTextureCache, 
                                          sourceImage: pixelBuffer, 
                                          textureAttributes: nil, 
                                          pixelFormat: MTLPixelFormatRGBA16Float, 
                                          width: width, 
                                          height: height, 
                                          planeIndex: 0, 
                                          textureOut: &cvTexture)

let texture = CVMetalTextureGetTexture(cvTexture)

// In Obj-C, release CVMetalTextureRef in Metal command buffer completion handlers
```
swift apps should use a strong reference for receiving a texture.  But in objc, ensure metal is done with texture before releasing the ref.

That's all folks!

# Wrap up
you learned about avplayer, avplayerviewcontroller
avplayerlayer
real-time effects during playback
applying real-time effects using CIFilter as well as metal shaders.

[[Edit and Playback HDR Video with AVFoundation]]
[[Explore EDR on iOS]]
[[Display EDR content with Core Image, Metal, and SwiftUI]]
https://developer.apple.com/av-foundation/

