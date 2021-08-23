Diversity in usecases creates tradeoffs between quality and performance for taking photos.

#avkit 

# How to take a photo on iOS?
* AVCaptureSession
* AVCaptureDevice
* AVCaptureDeviceInput
* AVCapturePhotoOutput
* AVCaptureConnection

call `.capturePhoto(with: settings, delegate: self)`

[[Advances in iOS Photography - 16]]

Let's see how high-quality photos can be taken. 

# High-quality photos
Usually, you used still image stabilization API.  Now we have better techniques.

In iOS 13, we introduced quality priotization.  We haven't talked about this previously.

Three levels
* .speed
* .balanced
* .quality

Note that this is only a hint.  Ultimately, protocol will consider a variety of datapoints.  Might choose a different method for low-light for example.

`.photoProcessingtimeRange` indicates how long it will take to deliver a photo to a delegate.

Only take waht you need.  Some of these may make sessions simpler for us to construct.

Using two different levels in different situations.  Note that we cannot go beyond `maxQualityPrioritizion` or you will get an exception.  

# Algorithms mapping
* Different mappings for photo and video formats
| Prioritization | Photo formats                           | Video formats |
|----------------|-----------------------------------------|---------------|
| Speed          | WYSIWYG                                 | WYSIWYG       |
| Balanced       | Fast fusion algorithms                  | WYSIWYG       |
| Quality        | Slower, more powerful fusion algorithms | WYSIWYG       |

# Photo formats
* Photo centric
* Exclusive features - live photo, proraw, etc
* Highest resolution, limited frame rate
* Photo session preset
* Check `isHighestPhotoQualitySupported`

# Video formats
* Video centric
* Lower resolution, high frame rate if needed
* Non-photo session preset
* Check `isHighestPhotoQualitySupported` is `false`

Why not algorithms for video?
If we used algoriths, we might introduce degredations in people doing raw input.  e.g., if you're doing ARKit, snapping a photo that does expensive processing might drop frames.

* Prevent experience degredations
* Avoid frame drops, preview feed interruptions, etc.
* Responsive APIs

# High photo quality using video formats!
# What do you get for each priorization?


| Prioritization | Results                                     | Frame drops or interruptions |
|----------------|---------------------------------------------|------------------------------|
| Speed          | WYSIWYG                                     | No                           |
| Balanced       | Significantly improved, fast algorithms     | No                           |
| Quality        | Significantly improved, powerful algorithms | Device-dependent             |

# Where is available?
* All iPhones starting at iPhone XS

| Resolution | Frame rates   |
|------------|---------------|
| 1280x720   | 30 and 60 fps |
| 1920x1080  | 30 and 60 fps |
| 1920x1440  | 30 fps        |
| 4k         | 30 fps        |

`format.isHighPhotoQualitySupported`.  For formats where this is true, it's a video format that supports this.

Keep in mind is *high* is different than is **highest**.  Latter tells you it's a photo format.

# Any code change?
Maybe.  

Using `AVCapturePhotoOutput` with `.speed`, you get a one-liner change to get better photos.

Using `.quality`?
Simply recompile

# Caveats
* AVCaptureMultiCamSession is **not** supported
* AVCaptureStillImageOutput is **not** supported
* Photo might look different from video frame
	* If you need them to match, use `.speed`

# Wrap up
* Be aware of quality and speed trade-offs
* Use appropriate quality prioritizations
* Amazing image quality for virtually (and perhaps literally) no work


