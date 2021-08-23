#videotoolbox

Introduce a new encoding mode for low-latency encoding.

# Optimize for real-time video apps
1.  Latency
2.  Interoperability
3.  Efficiency
4.  Video quality
5.  Error resilience

# Low-latency video encoding overview
Latency in real-time application
* Processing time
* Network transmission time
* Frame reordering is eliminated
* Rate controlelr minimizes the network delay

Can reduce delay by up to 100ms for 720p.  Critical for video conferencing.

* Real-time communication
* Hardware accelerated encoding
* Supported video codec: H.264
* Suported platforms: iOS, macOS
# Getting it done with Video Toolbox
## VTCompressionSession
1.  Create session
2.  Configure session
3.  Encode frames

Only change we need is in session creation.

```objc
CFMutableDictionaryRef encoderSpecification =
            CFDictionaryCreateMutable(kCFAllocatorDefault, 0, NULL, NULL);

CFDictionarySetValue(encoderSpecification,
                     kVTVideoEncoderSpecification_EnableLowLatencyRateControl,
                     kCFBooleanTrue)

VTCompressionSessionRef compressionSession;

OSStatus err = VTCompressionSessionCreate(kCFAllocatorDefault, 
                                          width, 
                                          height,
                                          kCMVideoCodecType_H264, 
                                          encoderSpecification,
                                          NULL, 
                                          NULL, 
                                          outputHandler, 
                                          NULL,
                                          &compressionSession);
```

## Configuration
Same as usual
# More features in low latency mode
## New profiles
We added 2 new profiles to the pipeline.

* Baseline
* Main profile
* High profile

new:
* Constrained baseline profile (CBP)
* Constrained high profile (CHP)

Check decoder capabilities to  know which profile should be used.

```objc
// Request CBP

VTSessionSetProperty(compressionSession, 
                     kVTCompressionPropertyKey_ProfileLevel, 
                     kVTProfileLevel_H264_ConstrainedBaseline_AutoLevel);

// Request CHP

VTSessionSetProperty(compressionSession, 
                     kVTCompressionPropertyKey_ProfileLevel, 
                     kVTProfileLevel_H264_ConstrainedHigh_AutoLevel);
```


## Temporal scalability
Helpful for conferencing

2 receivers with different bandwidth.  Normally, the sender needs to send two different bitstreams.  Maybe you want to use a single bitstream that can be divided into 2 layers.

Normally, each frame uses the previous frame as its reference.

We can pull half the frames into another layer, and change the reference so that only frames in the original layer are used for prediction.  Base layer, enhancement layer.

One receipient gets base layer, the other gets both layers.

50% of input framerate, 60% of target bitrate.  Only require the encoder to encode a single bitstream, which is more power-efficient for multiparty conferencing.

Error resilience: No dependency on enhancement layer frames.  So if one or more of these are dropped, other frames won't be affected.  Makes the whole session more robust.

Set base layer frame rate fraction to 0.5.  So half of frames are assigned to base layer.

`IsDependedOnByOthers` i true for base layer, false for enhancement layer.

Can set target bitrate for each layer.  `BaseLayerBitRateFraction` is a percentage of `AverageBitRate`.  If not set, we use 60% by default.  We recommend 60-80%.

## Max frame quanization parameter
Frame QP is used to regulate image quality and data rate.  Low frame QP generates a high-quality image.

* High frame QP generates an image in low quality with smaller size.
* Encoder adjusts frame QP based on various factors
	* complexity
	* framerate
	* video motion

In some cases, where the client has a specific requirement for video quality, can control max QP.  Encodr will always choose frame QP that is smaller than this limit.

Rate control wil be achieved by frame dropping
Transmitting screen content over a poor network

Maybe sacrifice framerate to send sharp images.  Setting max QP can meet this requierment.

### interface
`MaxAllowedFrameQP`
The following frame QP will be capped
Must be 1-51


## Long-term reference

When receiver client detects a frame loss, it can request a new frame.  Normally encoder will encode a keyframe, but it's usually quite large.  Takes a long time to get to the receiver.  Since network is already poor, it can cause issues.

Decide LTR.
Sender client sends an LTR frame and requests acknowledgment.
Ack sent back to encoder.
Encoder knows which LTR frame has been received so it can use that and make an LTR-P.  LTR-P is smaller than a keyframe.  

Requires application layer cooperation.  
* `EnableLTR`
* `RequireLTRAcknowledgmentToken`
* `AcknowledgedLTRTokens`
	* More than one acknowledgment can come at a time
* `ForceLTRRefresh`
	* Request to encode LTR-P
	* A key frame is necoded if no acknowledged LTR available

# Wrap up
* Low latency encoding in Video Toolbox
* Using `VTCompressionSession` for low latency encoding
* Other features for real-time video applications

https://developer.apple.com/documentation/videotoolbox


