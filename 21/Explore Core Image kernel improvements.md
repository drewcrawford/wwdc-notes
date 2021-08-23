#coreimage 

# Benefits of writing Core Image Kernels in Metal
* All the great features of CIKernels
* Reduced runtime compile time
* Advanced language features for performance
* Great syntax highlighting and inline error checking

## Two ways to add Metal CIKernels
* Extern CIKernels
	* Requires functions to be `extern "C"`
	* Requires custom Metal build flags
* Stitchable CIKernels
	* Requires functions to be `[[stitchable]]`.  Uses Metal Dynamic Libraries.

1.  Project config
2.  Add source file
3.  Write kernel
4.  Init and apply Kernel

### Extern CIKernels
Add build rules to your targets for `.ci.metal`
```
xcrun metal -w -c -fcikernel "${INPUT_FILE_PATH}" -o "${SCRIPT_OUTPUT_FILE_0}"
```

Output files
`${METAL_LIBRARY_OUTPUT_DIR}/${INPUT_FILE_BASE}.metallib`

Now a build rule for `*.ci.air`

```
xcrun metallib -cikernel "${INPUT_FILE_PATH}" -o "${SCRIPT_OUTPUT_FILE_0}"
```

Output files
```
${METAL_LIBRARY_OUTPUT_DIR}/${INPUT_FILE_BASE}.metallib
```

Now just use `.ci.metal` sources.

```cpp
// MyKernels.ci.metal
#include <CoreImage/CoreImage.h> // includes CIKernelMetalLib.h

using namespace metal;

extern "C" float4 myKernel (coreimage::sample_t s, 
                            float param, 
                            coreimage::destination dest) 
{
  float4 result = s;
  
  // Example code to create striped pattern
	float diagLine = dest.coord().x + dest.coord().y;
	float stripe   = fract(diagLine/20.0 + param*2.0);
  
  // Color range check
	if((stripe > 0.5) && ((s.r > 1) || (s.g > 1) || (s.b > 1)))
		result = float4(2.0, 0.0, 0.0, 1.0);
  
	return result;
}
```

[[HDR editing and playback using AVFoundation]]

```swift
class MyFilter: CIFilter {
    var inputImage: CIImage?
	var inputParam: Float = 0.0
    static var kernel: CIColorKernel = { () -> CIColorKernel in 
	  let url = Bundle.main.url(forResource: "MyKernels", 
                              withExtension: "ci.metallib")!
      let data = try! Data(contentsOf: url)
	  return try! CIColorKernel(functionName: "MyKernel", 
                              fromMetalLibraryData: data)
	}()
	override var outputImage : CIImage? {
      get { guard let input = inputImage else { return nil }
		return MyFilter.kernel.apply(extent:input.extent, 
                                 arguments:[input, inputParam]) }
	}
}
```

### Stitchable CIKernels.
* Add build setting to your targets so Metal linker links against CoreImage.
* `-framework CoreImage`.
* No custom build rule required
* No suffix required
* By default xcode builds everything into a metallib

```cpp
// MyKernels.ci.metal
#include <CoreImage/CoreImage.h> // includes CIKernelMetalLib.h

using namespace metal;

[[stitchable]] float4 myKernel (coreimage::sample_t s, 
                                float param, 
                                coreimage::destination d) 
{
  float4 result = s;
  
  // Example code to create striped pattern
	float diagLine = dest.coord().x + dest.coord().y;
	float stripe   = fract(diagLine/20.0 + param*2.0);
  
  // Color range check
	if((stripe > 0.5) && ((s.r > 1) || (s.g > 1) || (s.b > 1)))
		result = float4(2.0, 0.0, 0.0, 1.0);
  
	return result;
}
```

```swift
class MyFilter: CIFilter {
    var inputImage: CIImage?
	var inputParam: Float = 0.0
    static var kernel: CIColorKernel = { () -> CIColorKernel in 
	    let url = Bundle.main.url(forResource: "default", 
                                withExtension: "metallib")!
		let data = try! Data(contentsOf: url)
		return try! CIColorKernel(functionName: "MyKernel", fromMetalLibraryData: data)
	}()
	override var outputImage : CIImage? {
      get { guard let input = inputImage else { return nil }
		return MyFilter.kernel.apply(extent:input.extent, arguments:[input, inputParam]) }
	}
}
```

## Other benefits of `[[stitchable]]`
* Can link against other Metal libraries
* Can have integer or unsigned integer vector type parameters
* Can be compiled from source at runtime
	* Probably don't use this, but there are some applications

## Prerequisites for `[[stitchable]]` kernels
* MSL 2.4 – `[[stitchable]]` attribute.
* Metal dynamic libraries – for linking Core Image Metal classes

[[Discover compilation workflows in Metal]]

Only supported on some graphics devices

A11+
Apple silicon macs
Intel macs with AMD Navi and Vega GPUs



# Wrap up
* build rules
* write kernel source
* use from swift

https://developer.apple.com/documentation/coreimage

