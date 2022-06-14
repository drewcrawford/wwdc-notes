Four parts.
# Introduction and terminology
* SDR - standard dynamic range.  From 0-1.
* EDR - recommended way to represent RGB colors beyond the normal range.
* While values >1 are allowed, values >headroom will be clipped.
	* Display peak nits / SDR nits
	* Can vary between displays or as ambient conditions change
[[Explore EDR on iOS]]

Sources
* File formats such as float TIFF and OpenEXR
* Frames from HDR video files
* Metal GPU rendered scenes
* ProRAW image files
[[Capture and process ProRAW images]]

# Core Image with metal and SwiftUI
New sample project
combines Core Image and MTKView in a SwiftUI multiplatform app.
Demo

App displays an animated checkerboard CIImage as a proxy for content.
common codebase across macOS, iOS, iPadOS platforms.

1.  MetalView
2. ViewRepresentable to bridge between SwiftUI and platform-specific MTKView.
3. MTKView delegate is a Renderer type.
	4. MTLCommandQueue
	5. CIContext
	6. draw()
	7. imageProvider
8. ContentView is the imageProvider.

```swift
// Metal View

struct MetalView: ViewRepresentable {
    
    @StateObject var renderer: Renderer
    
    func makeView(context: Context) -> MTKView {
        let view = MTKView(frame: .zero, device: renderer.device)
       
        view.delegate = renderer

        // Suggest to Core Animation, through MetalKit, how often to redraw the view.
        view.preferredFramesPerSecond = 30
       
        // Allow Core Image to render to the view using Metal's compute pipeline.
        view.framebufferOnly = false
        
       return view
    }
```

preferredFramesPerSecond This app sets `view.preferredFramesPerSecond` so the view drives draw events
View itself drives timing of draw events.
Calls renderer
calls contentprovider
etc

Editing apps set `view.enableSetNeedsDisplay` so controls drive draw events.
When a control is moved, the `updateView` method should be called, then the view's delegate will be called to draw once.  Each draw will ask the content provider to create the CIImage.

Video apps can do the same so arrival of frames drives draw events.

draw method

CIImages are rendered in pixels, not points.  scale can change if the view is moved to a different display.

```swift
// Renderer

func draw(in view: MTKView) {
  if let commandBuffer = commandQueue.makeCommandBuffer(),
     let drawable = view.currentDrawable {
      // Calculate content scale factor so CI can render at Retina resolution.
  #if os(macOS)
      var contentScale = view.convertToBacking(CGSize(width: 1.0, height: 1.0)).width
  #else
      var contentScale = view.contentScaleFactor
  #endif

      let destination = CIRenderDestination(width: Int(view.drawableSize.width),
                          height: Int(view.drawableSize.height), 
                          pixelFormat: view.colorPixelFormat,
                          commandBuffer: commandBuffer,
                          mtlTextureProvider: { () -> MTLTexture in
                                   return drawable.texture
                          })
       
      let time = CFTimeInterval(CFAbsoluteTimeGetCurrent() - self.startTime)

      // Create a displayable image for the current time.
      var image = self.imageProvider(time, contentScaleFactor)

      image = image.transformed(by: CGAffineTransform(translationX: shiftX, y: shiftY))
      image = image.composited(over: self.opaqueBackground)
                
      _ = try? self.cicontext.startTask(toRender: image, from: backBounds,
                                             to: destination, at: CGPoint.zero)
```

```swift
// ContentView

import CoreImage.CIFilterBuiltins

init(struct ContentView: View {
    var body: some View {
       // Create a Metal view with its own renderer.
       let renderer = Renderer(
            imageProvider: { (time: CFTimeInterval, scaleFactor: CGFloat) -> CIImage in
            
            var image: CIImage

            // create image using CIFilter.checkerboardGenerator...

            return image
        })
        MetalView(renderer: renderer)
    }
}
```


# Adding support for EDR headroom
How to add support for EDR.

1. Initialize view for EDR
2. when rendering, calculate headroom
3. when building image, use headroom
```swift
if let caMtlLayer = view.layer as? CAMetalLayer  {
    caMtlLayer.wantsExtendedDynamicRangeContent = true
    view.colorPixelFormat = MTLPixelFormat.rgba16Float
    view.colorspace = CGColorSpace(name: CGColorSpace.extendedLinearDisplayP3)
}
```

```swift
let screen = view.window?.screen;
#if os(macOS)
     let headroom = screen?.maximumExtendedDynamicRangeColorComponentValue ?? 1.0
#else
     let headroom = screen?.currentEDRHeadroom ?? 1.0
#endif
     var image = self.imageProvider(time, contentScaleFactor, headroom)
```

Headroom is a dynamic property which may vary with ambient conditions or display brightness.
Finally, we use the headroom.

```swift
imageProvider: { (time: CFTimeInterval, scaleFactor: CGFloat,
                              headroom: CGFloat) -> CIImage in
           var image: CIImage

           // Use CIFilters to create image for time / scale / headroom / ...
           return image
        })
```

we did those steps.

# Using CIFilters with EDR
Over 150 filters support EDR.
ex, CIColorControls and CIExposureAdjust can alter brightness, hue, saturation, contrast of images with EDR colors.  Several filters such as gradient filters, can generate images given EDR color parameters.

* CIAreaLogarithmicHistagram => produce a histogram
* many more

Most built-in filters work because CI's working space is unclamped linear.  This allows RGBA values outside 0-1.
Check if a built-in filter supports edr.  Ask filter's attributes for its categories and check if that array contains `kCICategoryHighDynamicRange`.
Now can quicklook a filter instance in Xcode.
Infinite variety of effects that your app can apply.  Today we will add a ripple effect with a bright specular reflection.

```swift
let ripple = CIFilter.rippleTransition()
ripple.inputImage = image
ripple.targetImage = image
ripple.center = CGPoint(x: 512.0, y: 384.0)
ripple.time = Float(fmod(time*0.25, 1.0))
ripple.shadingImage = shading
image = ripple.outputImage
```

How to create shading image that creates a specular highlight.  To get even better performance, we can generate procedurally.

```swift
let gradient = CIFilter.linearGradient()
let w = min( headroom, 8.0 ) //based on the headroom, but limited to some maximum.
gradient.color0 = CIColor(red: w, green: w, blue: w, 
                          colorSpace: CGColorSpace(name: CGColorSpace.extendedLinearSRGB)!)!
gradient.color1 = CIColor.clear
//place specular in upper left
gradient.point0 = CGPoint(x: sin(angle)*90.0 + 100.0, y: cos(angle)*90.0 + 100.0)
gradient.point1 = CGPoint(x: sin(angle)*85.0 + 100.0, y: cos(angle)*85.0 + 100.0)
let shading = gradient.outputImage?.cropped(to: CGRect(x: 0, y: 0, width: 200, height: 200))
```

Best to use bright pixels in moderation.  Less is more.

Feel free to experiment with other EDR filters.

# CIColorCube
* traditionally this filter used to apply "looks" to SDR images
* usually, this is SDR only.
* One way to avoid this is to use an EDR colorspace with `CIColorCubeWithColorSpace`.
* This gives the best quality
* requires the cube data to be valid over EDR range
* may need to increase the cube dimensions
Instead, you may want to ocntinue to use SDR data for EDR images.
* tell the filter to extrapolate the SDR cube data

```swift
let f = CIFilter.colorCubeWithColorSpace()
f.cubeDimension = 32
f.cubeData = sdrData
f.extrapolate = true //magic
f.inputImage = edrImage
let edrResult = f.outputImage
```

# Best practices for custom CIKernels
Avoid math that clamps RGB to 0..1
Dont' use *alpha* that is >1 => **UB**
# Wrap up
* How to add supprot for EDR headroom to a core image swiftui application
* How to use CIFilters to create and modify EDR images

https://developer.apple.com/documentation/coreimage/generating_an_animation_with_a_core_image_render_destination
https://developer.apple.com/documentation/coreimage


