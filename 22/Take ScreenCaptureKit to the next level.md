#screencapturekit 

Advanced topics in ScreenCaptureKit.  Exciting new APIs.

At the heart of screen sharing applications such as zoom, google meet, shareplay.  Popular game streaming services.  New norm of how we work, study, collaborate, or socialize.

# ScreenCaptureKit
Brand new high performance screencapture framework built with a powerful featureset.  The rich set of features includes
* customizable screen content capture
* High resolution, frame rate, and performance
* Dynamic stream property control
* GPU memory backed
* GPU accelerated with reduced CPU usage
* Captures both video and audio
Please see intro session,
[[Meet ScreenCaptureKit]]

# Capture a single window
* set up single window filter
* window-based capture output
* Use per-frame metadata
* Display captured window

Use a single window filter and initialize the filter with just one window.  ScreenCaptureKit audio capture policy always works as the app level.  When a single window filter is used, all the audio content is captured.  Even from windows that are not present in the ivdeo output.

```swift
// Get all available content to share via SCShareableContent
let shareableContent = try await SCShareableContent.excludingDesktopWindows(false, 
       onScreenWindowsOnly: false)

// Get window you want to share from SCShareableContent
guard let window : [SCWindow] = shareableContent.windows.first( where: 
                                    { $0.windowID == windowID }) else { return } 

// Create SCContentFilter for Independent Window
let contentFilter = SCContentFilter(desktopIndependentWindow: window)

// Create SCStreamConfiguration object and enable audio capture
let streamConfig = SCStreamConfiguration()
streamConfig.capturesAudio = true

// Create stream with config and filter
stream = SCStream(filter: contentFilter, configuration: streamConfig, delegate: self)
stream.addStreamOutput(self, type: .screen, sampleHandlerQueue: serialQueue)
stream.startCapture()
```

## Output frame rate
source display, stream output.
stream filter includes a single safari window.
Stream output includes the live content from the single safari window and is updating at same cadence as the source window, up to the source display's native framerate.  for example, when a source window is comfortable updating on 120hz display, stream output can also achieve up to 120fps.

You may wonder what happens when window resizes.  Keep in mind that frequently-changing the screen's output dimensions can lead to additional allocations and is not recommended.

Stream output is mostly fixed, does not resize with the source window.  ScreenCaptureKit always performs hardware scaling a tthe captured window so it never exceeds frame output as the source window resizes.

What bout covered windows?  When a source window is occluded or partially occluded, stream output always includes the window's full contents.

This aplso applies tot he case where the window is completely offscreen or moves to other displays.  When soruce window is minimized, stream output is paused and it resumes when the source window is no longer minimized.

## audio output
Two safari windows.  Left is captured.

Video output includes just the first window, and the audio track from both windows are included.

## Per-frame stream output and metadata
* IOSurface rendering captured frame
* additional metadata

* dirty rects => indicatre where new content is from the previous frame.  Instead of always encoding the entire frame, we can simply use dirty rects to only encode and transmits the regions where soetmhing changed, and copy onto previous frame on the receiver isde.
* contect rect
* content scale
* Scale factor


With the stream app running, your app receives a frame updates whenever there is a new frame available.
```swift
// Get dirty rects from CMSampleBuffer dictionary metadata

func streamUpdateHandler(_ stream: SCStream, sampleBuffer: CMSampleBuffer) {
    guard let attachmentsArray = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, 
                                                          createIfNecessary: false) as? 
                                                         [[SCStreamFrameInfo, Any]], 
        let attachments = attachmentsArray.first else { return }
        
        let dirtyRects = attachments[.dirtyRects]    
    }
}

// Only encode and transmit the content within dirty rects
```

## Content rect and content scale
Source window to be captured is on the left.  Stream output on the right.

Since a window can be resized, source window's native backing surface might not match stream output dimensions.  In this example, captured window has different aspect ratio from frame's output.

Captured window scales down to fit into the output.  

A content rect indicates ther egion of interest of the captured ocntent on the stream output.  Contents cale indicates how much the content is sacled to fit.  Here the cpatured safari window is scaled down by 0.77 to fit inside the frame.

Can use this to display as closely as possible.
1.  Crop content from its output using the content rect.
2. Scale the content back up by dividing the content scale.
3. Captured ocntent is scaled to match 1:1

A display's scale factor indicates a scale ratio between a display or windows' logical point size, and the backing surface pixel size.

scale factor = 2, 2x mode.  1 point on screen is 4px in backing surface.  Window can be moved from a retina display.  Non-retina display with scale factor 1.

In this example, a window is being captured from a retina display on the left, with a scale factor of 2.  And to be displayed on a non-retina display on right.  If the captured window is displayed as-is, it will be very big.

Always check the scale factor from the frame's metadata against the scale factor of the target display.  When there's a mismatch,s cale the size of the captured ocntent appropriately.
```swift
/* Get and use contentRect, contentScale and scaleFactor (pixel density) to convert the captured window back to its native size and pixel density */

func streamUpdateHandler(_ stream: SCStream, sampleBuffer: CMSampleBuffer) {

    guard let attachmentsArray = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, 
                                                          createIfNecessary: false) as? 
                                                         [[SCStreamFrameInfo, Any]], 
        let attachments = attachmentsArray.first else { return }

        let contentRect = attachments[.contentRect]
        let contentScale = attachments[.contentScale]
        let scaleFactor = attachments[.scaleFactor]

        /* Use contentRect to crop the frame, and then contentScale and 
        scaleFactor to scale it */

    }
}
```

Use these metadata to crop and scale the captured content to display it correctly.

To recap, a single window filter always includes the full window.
Even when the source window is offscreen or occluded.
It's display and space independent.
Output is always at the top left `0,0`
Does *not* include pop-up or child window
Use metadata to display
Audio includes tracks from the entire app

# Add content to capture
## Capture a display
* capture a display with windows
* or apps
* Video vs audio filtering rules

No windows are captured by default.  Add Safari window and keynote window.
```swift
// Get all available content to share via SCShareableContent
let shareableContent = try await SCShareableContent.excludingDesktopWindows(false, 
                                                            onScreenWindowsOnly: false)

// Create SCWindow list using SCShareableContent and the window IDs to capture
let includingWindows = shareableContent.windows.filter { windowIDs.contains($0.windowID)}

// Create SCContentFilter for Full Display Including Windows
let contentFilter = SCContentFilter(display: display, including: includingWindows)

// Create SCStreamConfiguration object and enable audio
let streamConfig = SCStreamConfiguration()
streamConfig.capturesAudio = true

// Create stream
stream = SCStream(filter: contentFilter, configuration: streamConfig, delegate: self)
stream.addStreamOutput(self, type: .screen, sampleHandlerQueue: serialQueue)
stream.startCapture()
```

With the stream up and running, let's take alook at the stream's output.  Windows moved off screen or otanother display are clipped.

If a window is moved off screen, it will be removed from stream output.
New windows from same 

New window is not in the filter.  Same rule also applies to child or popup windows, which do not show up in the steram's output.

Ensure that child windows are included automatically, use a dispaly-based filter with included apps.  In this example, adding the safari and keyntoe apps to the filter ensures that the audio and video output from all the windows and soundtracks are included in output.

Window exception filters are a powerful wayu of exlcuding specific windows from your output when the filter is specified as the display with included apps.

Ex, a signle safari window is removed from the output.  ScreenCaptureKit enables audio captuer at the app level, so exlcuding audio from a single window is the equivalent to removing audio tracks from all safari apps.  Although the stream's visual output still includes a safari window, all the subtracks from safari apps are removed and the audio output includes just the soundtrack from keynote.
```swift
// Get all available content to share via SCShareableContent
let shareableContent = try await SCShareableContent.excludingDesktopWindows(false, onScreenWindowsOnly: false)

/* Create list of SCRunningApplications using SCShareableContent and the application 
IDs you’d like to capture */
let includingApplications = shareableContent.applications.filter { 
    appBundleIDs.contains($0.bundleIdentifier)
}

// Create SCWindow list using SCShareableContent and the window IDs to except
let exceptingWindows = shareableContent.windows.filter { windowIDs.contains($0.windowID) }

// Create SCContentFilter for Full Display Including Apps, Excepting Windows
let contentFilter = SCContentFilter(display: display, including: includingApplications,
                           exceptingWindows: exceptingWindows)
```


# Remove content from capture
Mirror hall effect.  Even during full display share, it's common for screen sharing applications to remove its own windows, preview, participant camera view, etc.  Notification windows, etc.

Quickly remove ocntent from display capture.  Ane xclusion-based display filter captures all the window by default.  Can then start to remove individual windows or apps by adding them to the exlcusion filter.

```swift
// Get all available content to share via SCShareableContent
let shareableContent = try await SCShareableContent.excludingDesktopWindows(false, 
                                         onScreenWindowsOnly: false)

/* Create list of SCRunningApplications using SCShareableContent and the app IDs 
you’d like to exclude */
let excludingApplications = shareableContent.applications.filter { 
    appBundleIDs.contains($0.bundleIdentifier)
}

// Create SCWindow list using SCShareableContent and the window IDs to except
let exceptingWindows = shareableContent.windows.filter { windowIDs.contains($0.windowID) }

// Create SCContentFilter for Full Display Excluding Windows
let contentFilter = SCContentFilter(display: display, 
                        excludingApplications: excludingApplications, 
                        exceptingWindows: exceptingWindows)
```

audio will be removed as well.

# Stream configuration
* configuration propperties
* screen captuer and streaming
* window picker with live preview

## Configuration properties
* output dimensions
* source and destination rects
* colors and pixel format
* cursor

### output dimensions
doesn't always match the output dimensions.  When this mismatch happens while capturing a full display, there will be pillars/letterbox in stream output.

## source and destination rects
Can define region to capture from.  Rendered/scaled into destination rect.

### color/pixel formats
hardware-accelerated conversions.  Common BGRA and YuV are supported.  Please visit our developer page for the full list.

### Cursor
Prerendered into frame.  Applies to all system cursors, even custom cursors.

### Frame rate
You can use minimum frame interval to control.
capped by content's native frame rate

### Queue depth
Number of IOSurfaces in the surface pool
more surfaces => better frame rate, higher system memory usage, latency tradeoff
valid range is 3 to 8

When surface 1 is complete, SCKit sends surface 1 to your app.  Your app is processing/holding 1, while SCKit is rendering to surface 2.  

New surfaces may pile up, etc.  In this case, more surfaces in the pool can potentially lead to higher latency.  The number of surfaces left in the pool for SCKit to use is the queue depth - surfaces held by your app.

If your app continues to hold surfaces, SCKit will incur frame loss.  

1.  To avoid frame delay: A < minimumFrameInterval
2. To avoid frame loss: B < MinimumFrameInterval * (QueueDepth - 1)

When we run out of surfaces, we miss new frames.


## Screen capture and streaming
Animation and video.  High FPS, low resolution.
Static text: low FPS, high resolution.
Can live-adjust configuration based on content being shared and network conditions.
```swift
let streamConfiguration = SCStreamConfiguration()

// 4K output size
streamConfiguration.width  = 3840
streamConfiguration.height = 2160

// 60 FPS
streamConfiguration.minimumFrameInterval = CMTime(value: 1, timescale: CMTimeScale(60))

// 420v output pixel format for encoding
streamConfiguration.pixelFormat = kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange

// Source rect(optional)
streamConfiguration.sourceRect = CGRectMake(100, 200, 3940, 2360)

// Set background fill color to black
streamConfiguration.backgroundColor = CGColor.black 

// Include cursor in capture 
streamConfiguration.showsCursor = true

// Valid queue depth is between 3 to 8
streamConfiguration.queueDepth = 5

// Include audio in capture
streamConfiguration.capturesAudio = true
```

All stream ocnfiguration you just saw... can be dynamically changed on the fly without recreating the stream.  ex live adjust some properties such as
* output dimension
* dynamic frame rate
* filters

```swift
// Update output dimension down to 720p
streamConfiguration.width  = 1280
streamConfiguration.height = 720

// 15FPS
streamConfiguration.minimumFrameInterval = CMTime(value: 1, timescale: CMTimeScale(15))

// Update the configuration
try await stream.updateConfiguration(streamConfiguration)
```

## Window picker with live preview
In the last example, I'd like to walk through building a window picker with live preview.  Here is an example of what a typical window picker looks like.

SCKit provides an efficient and high-performance solution for creating large number of thumbnail-sized streams with live content updates, and it's simple to implement.

1.  1 stream per window with single window filter
2. set up configuration with thumbnail-sized 5fps, BGRA, queue depth 3, no cursor, no audio

```swift
// Get all available content to share via SCShareableContent
let shareableContent = try await SCShareableContent.excludingDesktopWindows(false, 
                                        onScreenWindowsOnly: true)

// Create a SCContentFilter for each shareable SCWindows
let contentFilters = shareableContent.windows.map { 
    SCContentFilter(desktopIndependentWindow: $0) 
}

// Stream configuration
let streamConfiguration = SCStreamConfiguration()

// 284x182 frame output
streamConfiguration.width  = 284
streamConfiguration.height = 182
// 5 FPS
streamConfiguration.minimumFrameInterval = CMTime(value: 1, timescale: CMTimeScale(5))
// BGRA pixel format for on screen display
streamConfiguration.pixelFormat = kCVPixelFormatType_32BGRA
// No audio
streamConfiguration.capturesAudio = false
// Does not include cursor in capture 
streamConfiguration.showsCursor = false
// Valid queue depth is between 3 to 8

// Create a SCStream with each SCContentFilter
var streams: [SCStream] = []
for contentFilter in contentFilters {
    let stream = SCStream(filter: contentFilter, streamConfiguration: streamConfig, 
                        delegate: self)
    try stream.addStreamOutput(self, type: .screen, sampleHandlerQueue: serialQueue)
    try await stream.startCapture()
    streams.append(stream)
}
```

Create one stream for each window.  Add stream output, and then start the stream.  Append it to the stream list.

Demo.

# OBS studio adoption demo
* OBS studio is popular live streaming software
* Similar to existing implementation
* Improved performance, lower overhead
## Demo
* old: 7fps
* new: 60fps

up to 15% decrease in memory usage with a single window capture in OBS
50% decrease in cpu usage with a single window capture in OBS


# Wrap up
* advanced screen content filters
* stream configurations
* how to use per-frame metadata
* best practices
* OBS adoption demo
* 





* https://developer.apple.com/forums/tags/wwdc2022-10155
* https://developer.apple.com/forums/create/question?&tag1=376030&tag2=373030
* https://developer.apple.com/documentation/screencapturekit
* https://developer.apple.com/documentation/screencapturekit/capturing_screen_content_in_macos

