Discover how area mode for Object Capture enables new 3D capture possibilities on iOS by extending the functionality of Object Capture to support capture and reconstruction of an area. Learn how to optimize the quality of iOS captures using the new macOS sample app for reconstruction, and find out how to view the final results with Quick Look on Apple Vision Pro, iPhone, iPad or Mac. Learn about improvements to 3D reconstruction, including a new API that allows you to create your own custom image processing pipelines.

### Data Loading API - load Sample and Mask - 8:19

```swift
func loadSampleAndMask(file: URL) -> PhotogrammetrySample? { 
    do { 
        var sample = try PhotogrammetrySample(contentsOf: file) 
        sample.objectMask = try loadObjectMask(for: file) 
        return sample 
    } 
    catch { 
        return nil 
    } 
}
```

### Data Loading API - create custom photogrammetry Session - 9:15

```swift
func createCustomPhotogrammetrySession(for images: [URL]) -> PhotogrammetrySession { 
    let inputSequence = images.lazy.compactMap { file in 
        return loadSampleAndMask(file: file) 
    } 
    return PhotogrammetrySession(input: inputSequence) 
}
```

# Resources
* https://developer.apple.com/documentation/RealityKit/building-an-object-reconstruction-app
* https://developer.apple.com/documentation/RealityKit/guided-capture-sample
