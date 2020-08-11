#metal #coreimage

# Why do you want to write custom `CIKernels` in metal?

* Reduce runtime compile time
* Advanced language features for performance
* Great syntax highlighting and syntax checking

1.  Add custom buidl rules to your project
2.  Add `.ci.metal` sources to your project
3.  Write your kernel
4.  Initialize `CIKernel`
5.  Apply kernel to create a new `CIImage`

## build rules
### compile step
process: `Source files with names matching:` `*.ci.metal`
Using: `Custom script:`

```bash
xcrun metal -c -I $MTL_HEADER_SEARCH_PATHS -fcikernel "${INPUT_FILE_PATH}" -o "${SCRIPT_OUTPUT_FILE_0}"
```

output files
`$(DERIVED_FILE_DIR)/$(INPUT_FILE_BASE).air`

### link step
Process: `Source files with names matching:` `*.ci.air`
Using: `Custom script:`

```bash
xcrun metallib -cikernel "${INPUT_FILE_PATH}" -o "${SCRIPT_OUTPUT_FILE_0}"
```

output files
`$(METAL_LIBRARY_OUTPUT_DIR/$(INPUT_FILE_BASE).metalllib`

## Add `.ci.metal` sources to your project
[[Edit and Playback HDR Video with AVFoundation]]

Note that CI kernels must be `extern "C"`.

https://developer.apple.com/metal/MetalCIKernelReference6.pdf


```swift
class HDRZebraFilter: CIFilter {
	var inputImage: CIImage?
	var inputTime: Float = 0.0
	
	static var kernel: CIColorKernel = { () -> CIColorKernel in
		let url = Bundle.main.url(forResource: "MyKernels", withExtension: "ci.metalllib")!
		let data = try! Data(contentsOf: url)
		return try! CIColorKernel(functionName: "HDRzebra", fromMetalLibraryData: data)
		
	}()
	
	outverride var outputImage: CIImage? {
		get {
			guard let input = inputImage else { return nil }
			return HDRZebraFilter.kernel.apply(extent: input.extent, arguments: [input,inputTime])
		}
	}
}
```

