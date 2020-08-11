#coreimage 

# Creating `CIContext`

* Only create one context per view
* Create your context with options:
```swift
let context =  CIContext(options: [
    .cacheIntermediates : false, //disabling lowers memory usage
    .name : ”MyAppView”
])
```
[[Discover Core Image Debugging Techniques]]

If you are mixing Core Image with other metal usage
* For example when a core image input or output is a `MTLTexture`,

create context with `MTLCommandQueue`:

```swift
let context =  CIContext(MTLCommandQueue : queue, options: […])
```
`CIContext` uses an internal `MTLQueue`.  Because all this work is done on different queues, the app must issue wait commands to get the correct results.

Solution is to create a CIContext with the same queue.  This allows app to remove the waits.

# Writing and applying `CIKernel`s

* Built-in `CIFilters` are optimized for Metal
* Updated documentation

```swift
func motionBlur(inputImage: CIImage) -> CIImage? {
    let motionBlurFilter = CIFilter.motionBlur()
    motionBlurFilter.inputImage = inputImage
    motionBlurFilter.angle = 0
    motionBlurFilter.radius = 20
    return motionBlurFilter.outputImage
}
```

[[Edit and Play Back HDR video with AVFoundation]]

```swift
// MyKernels.ci.metal
#include <CoreImage/CoreImage.h> // includes CIKernelMetalLib.h
using namespace metal;

//must be extern "C"
//color kernel -> returns float4 pixel
extern "C" float4 HDRZebra (coreimage::sample_t s,  //pixel from an input image.  Linear, premultipleid RGBA float4
float time, coreimage::destination dest)  //coordinate of pixel to return
{
	float diagLine = dest.coord().x + dest.coord().y;
	float zebra = fract(diagLine/20.0 + time*2.0);
	if ((zebra > 0.5) && (s.r > 1 || s.g > 1 || s.b > 1))
		return float4(2.0, 0.0, 0.0, 1.0);
	return s;
}
```
[[Build Metal-based Core Image Kernels with Xcode]]



# Choosing a View Class
Avoid Image View classes: `UIImageView`, `NSImageView`
Simplest option: `AVPlayerView`
More flexible: MetalKit

```swift
let videoComposition = AVMutableVideoComposition(
    asset: asset,      applyingCIFiltersWithHandler:
    { (request: AVAsynchronousCIImageFilteringRequest) -> Void in
        let filter = HDRZebraFilter()         filter.inputImage = request.sourceImage
        let output = filter.outputImage

        if (output != nil) {
            request.finish(with: output, context: myCtx)
        }
        else { request.finish(with: err) }
    }
)
```
Xcode previews for CI pipelines.
```swift
class MyView : MTKView {
    var context: CIContext
    var commandQueue : MTLCommandQueue
    
    override init(frame frameRect: CGRect, device: MTLDevice?) {
        let dev = device ?? MTLCreateSystemDefaultDevice()!
        context = CIContext(mtlDevice: dev, options: [.cacheIntermediates : false] )
        commandQueue = dev.makeCommandQueue()!
        
        super.init(frame: frameRect, device: dev)

        framebufferOnly = false  // allow Core Image to use Metal Compute
        colorPixelFormat = MTLPixelFormat.rgba16Float
        if let caml = layer as? CAMetalLayer {
            caml.wantsExtendedDynamicRangeContent = true
        }
    }
```
	
```swift
	func draw(in view: MTKView) {

        let size = self.convertToBacking(self.bounds.size)
        let rd = CIRenderDestination(width: Int(size.width),
                                     height: Int(size.height),
                                     pixelFormat: colorPixelFormat,
                                     commandBuffer: nil)
									 //this block strategy allows us to start enqueing work before requiring the previous frame to complete
                  { () -> MTLTexture in return view.currentDrawable!.texture }

        context.startTask(toRender:image, from:rect, to:rd, at:point)

        // Present the current drawable
        let cmdBuf = commandQueue.makeCommandBuffer()!
        cmdBuf.present(view.currentDrawable!)
        cmdBuf.commit()
   }
```
