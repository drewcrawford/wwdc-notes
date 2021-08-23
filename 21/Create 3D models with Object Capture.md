Turn real-world objects into 3d models using photogrammatry.

Object capture: Easily turn images of real-world objects into detailed 3d models.

1.  Take photos
2.  Copy images to mac which supports API
3.  Photogrammetry
4.  Model

Drop right into app or view in AR quicklook. 

Capture photos from all sides

* iPhone, iPad
* DSRL, mirrorless
* Drone

Need clear photos from all angles, evidently no depth or similar data is needed

* Scale recovery using stereo depth (iPhone)
* Uprighting using gravity (iPhone)

Swift API in RealityKit (macOS)
 * All apple silicon macs
 * Intel macs with 4GB AMD and 16GB RAM
* HelloPhotogrammetry: command-line sample app provided

Use directly on your folder of images to try yourself.

USDZ, USDA, or OBJ
* Reduced
* medium
* full
* raw

By selecting USDZ output at medium, can view in AR quicklook.

Object capture can support a variety of uses, from AR assets to film-ready assets.

# Getting started
* Setup
* Process

## Create session
PhotogrammetrySession => A fixed container of image samples

Session input sources.
* Folder of images (HEIC, JPG, PNG)
* Sequence of PhotogrammetrySample
	* Image
	* depth
	* gravity
	* metadata
	* segmentation mask

Process requests
Generate messages
* Output results
* status
* errors
```swift
import RealityKit

let inputFolderUrl = URL(fileURLWithPath: "/tmp/Sneakers/", isDirectory: true)
let session = try! PhotogrammetrySession(input: inputFolderUrl,
                                         configuration: PhotogrammetrySession.Configuration())
```

## Connect output stream
`AsyncSequence` of `Output` messages.
* Result of request
* Status
* warnings

Calls to `process()` will produce messages asynchronously
Stream active for entire session lifetime

* requestProgress messages periodically.  Drive progress bar etc.
* requestComplete => Results.  Model, bounding box, etc.
* requestError => instead.
* processingComplete => all queued requests are finished processing.

```swift
// Create an async message stream dispatcher task

async {
    do {
        for try await output in session.outputs {
            switch output {
            case .requestProgress(let request, let fraction):
                print("Request progress: \(fraction)")
            case .requestComplete(let request, let result):
                if case .modelFile(let url) = result {
                  print("Request result output at \(url).")
                }
            case .requestError(let request, let error):
                print("Error: \(request) error=\(error)")
            case .processingComplete:
                print("Completed!")
                handleComplete()
            default:  // Or handle other messages...
                break
            }
        }
    } catch {
       print("Fatal session error! \(error)")
    }
}
```


## Request models
* Model file (URL)
	* File: USDZ single file
	* Folder: USDA and OBJ, assets
* ModelEntity
* BoundingBox => can be adjusted to adjust reconstruction volume

### Detail
* Preview => only for interactive workflows
* Reduced
* medium
* full
* raw => Post production workflow

## how to
```swift
try! session.process(requests: [
    .modelFile("/tmp/Outputs/model-reduced.usdz", detail: .reduced),
    .modelFile("/tmp/Outputs/model-medium.usdz", detail: .medium)
])
```

Requesting all levels simultaneously in one call allows sharing computation and is faster than requesting them sequentially.

# Interactive workflow
Designed to allow several adjustments to be made on a preview model before the final reconstruction.

Setup/process step are the same.  Also requesting models are the same.  However, we do iteration.

1.  Request preview (detail = `.preview`)
2.  GUI review / Adjust geometry

Adjust capture volume to remove any unwanted geometry from the capture.  e.g. a pedastal.

Scale/translate/rotate the model.  Geometry property allows a capture volume and root transform to be provided before the model is generated.

Once we're happy with the cropped preview, we can select a full detail final render.  After some time, the full detial model is complete.  Now we can see the full detail of the model.

Model is saved in the output directory and ready to use.

That's all there is to getting started!
* Create a session from an input source
* Connect the output stream
* Request models at different detail levels
* Consider interactive preview API for GUI apps

# Best practices
* Capture
	* Object selection
	* Taking images
	* CaptureSample app
	* Capture demos
* Selecting the right output

## Object selection
Good object characteristics.
* Sufficient texture detail => textureless or transparent regions may lack detail
* Minimal reflective surfaces => try diffusing lighting when you scan
* Rigid (when flipping)
* Minimal fine structure => may need a high-resolution camera to recover detail

## Taking images
* Ensure the object is in focus
* Capture from all angles
* Flip to image all sides
* Get close to the object (fill view)
	* Use portrait or landscape
* Maintain overlap between images
* 20-200 images depending on the object

## CaptureSample app
SwiftUI demo, part of documentation.  Demonstrates how to take high-quality photos.  Manual, timed shutter mode.  Demonstrates how to use iPhone etc, save gravity data, view gallery, delete bad shots.

We recommend turntable capture to get the best results possible.  Light tent to avoid hard shadows.  

## Selecting the right output.

Reduced => 1 triangle, 1 memory, iOS ready
Medium => 2 triangles, 4 memory, iOS ready
Full => 3 triangles, 12 memory, not iOS ready.  Pro workflow ok
Raw => 4 triangles, 18 memory, not iOS ready, pro workflow ok, unbaked

* Reduced/medium: ideal for web
* Medium - complex assets and apps
* Both include pre-baked PBR materials (normal/AO/diffuse)

### Full
* maximum detial for pro workflows
* Includes 5 pre-baked PBR materials channels
	* diffuse
	* normal
	* roughness
	* AO
	* diffuse

### Raw
* Multiple high-resolution texture
* No baked material maps
* Designed for custom post-production pipelines
* Diffuse 1-16?

[[Create 3D Workflows with USD]]

Remember you can select multiple detail levels to cover current and future usecases.

# Wrap up
* iOS and macOS sample apps
* Sample data sets

[[Create 3D Workflows with USD]]
[[AR Quick Look, meet Object Capture]]

https://developer.apple.com/documentation/realitykit/creating_a_photogrammetry_command-line_app
https://developer.apple.com/documentation/realitykit/taking_pictures_for_3d_object_capture
https://developer.apple.com/documentation/realitykit/creating_3d_objects_from_photographs
https://developer.apple.com/documentation/realitykit/capturing_photographs_for_realitykit_object_capture
https://developer.apple.com/documentation/realitykit/photogrammetrysample
https://developer.apple.com/documentation/realitykit/photogrammetrysession

