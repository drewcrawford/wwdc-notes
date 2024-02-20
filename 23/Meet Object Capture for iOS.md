Discover how you can offer an end-to-end Object Capture experience directly in your iOS apps to help people turn their objects into ready-to-use 3D models. Learn how you can create a fully automated Object Capture scan flow with our sample app and how you can assist people in automatically capturing the best content for their model. We'll also discuss LiDAR data and provide best practices for scanning objects.

# What is object capture?
Cutting-edge CV technologies to create a lifelike 3d model.  from iamges at various angles.

Object Capture API used to reconstruct 3d model in a matter of minutes.  We've seen many apps leveraging object capture to produce high-quality 3d models.

Now do capturing and reconstruction on same device.

Demo.

# More objects with LiDAR

We now support low-texture objects leveraging lidar. ex chair with matte fabric.

Now we collect point data with lidar, so we can get 3d shape with enhanced coveraged and density.  

Challenging objects.
* reflective
* transparent
* too-thin structures

# Guided capture

Automatic capture
Capture feedback
Flipping objects

ensure that
* lighting
* speed
* distance
* field of view

flipping object.  Helps to achieve capturing the whole object.  

If your object is rigid, flipping is a good idea.
But deformable, might not be a good idea.
Texture-rich objects: recommended.  But avoid flipping objects that have geometric or repetitive textures, because we may have trouble.
Textureless can also be complicated.

We provide an API to suggest whether it is textured enough for flipping.

* 3 orientations
* diffuse lights
* visual overlap

Non-flippable
* recommend capturing from 3 heights
* Textured background

supported devices on iOS
iPhone 12 pro and later with lidar
iPad pro 2021 and later with lidar
api to check support




# iOS API

* ObjectCaptureSession
* ObjectCaptureView

Move around to ensure that the whole object is inside the box, etc.

* initializing
* ready
* detecting
* capturing
* finishing
* completed

### Instantiating ObjectCaptureSession - 10:03
```swift
import RealityKit
import SwiftUI 

var session = ObjectCaptureSession()
```


recommend storing inside a ground truth data model until completed?
### Starting the session - 10:25
```swift
var configuration = ObjectCaptureSession.Configuration()
configuration.checkpointDirectory = getDocumentsDir().appendingPathComponent("Snapshots/")

session.start(imagesDirectory: getDocumentsDir().appendingPathComponent("Images/"),
              configuration: configuration)
```

checkpoint - can be used later to speed up reconstruction process.


### Creating ObjectCaptureView - 10:50
```swift
import RealityKit
import SwiftUI

struct CapturePrimaryView: View {
    var body: some View {
        ZStack {
            ObjectCaptureView(session: session)
        }
    }
}
```

always displays the UI corresponding to the state session,e tc.

### Transition to detecting state - 11:20
```swift
var body: some View {
    ZStack {
        ObjectCaptureView(session: session)
        if case .ready = session.state {
            CreateButton(label: "Continue") { 
                session.startDetecting() 
            }
        }
    }
}
```

### Showing ObjectCaptureView - 11:36
```swift
var body: some View {
    ZStack {
        ObjectCaptureView(session: session)
    }
}
```

### Transition to capturing state - 12:04
```swift
var body: some View {
    ZStack {
        ObjectCaptureView(session: session)
        if case .ready = session.state {
            CreateButton(label: "Continue") { 
                session.startDetecting()
            }
        } else if case .detecting = session.state {
            CreateButton(label: "Start Capture") { 
                session.startCapturing()
            }
        }
    }
}
```

### Showing ObjectCaptureView - 12:27
```swift
var body: some View {
    ZStack {
        ObjectCaptureView(session: session)
    }
}
```


### Completed scan pass - 12:50
```swift
var body: some View {
    if session.userCompletedScanPass {
        VStack {
        }
    } else {
        ZStack {
            ObjectCaptureView(session: session)
        }
    }
}
```

`beginNewScanPassAfterFlip()` to scan the flip situation.

Otherwise, can capture at a different height.  Use `.beginNewScanPass()`.  Resets capture dial but session remains in the capturing state since bounding box has not changed.

### Transition to finishing state - 14:03
```swift
var body: some View {
    if session.userCompletedScanPass {
        VStack {
            CreateButton(label: "Finish") {
                session.finish() 
            }
        }
   } else {
        ZStack {
            ObjectCaptureView(session: session)
        }
    }
}
```

finishing - we're saving data.

errors -> such as the image directory becomes unavailable, etc.



### Point cloud view - 15:00
```swift
var body: some View {
    if session.userCompletedScanPass {
        VStack {
            ObjectCapturePointCloudView(session: session)
            CreateButton(label: "Finish") {
                session.finish() 
            }
        }
    } else {    
        ZStack {
            ObjectCaptureView(session: session)
        }
    }
}
```

can also display the point cloud as shown.  

Interact with the point cloud to preview from all angles.  Show in combiantion with text and buttons, etc.  

Now you can run reconstruction API on iOS.  Run image capture and reconstruction on the same device.
### Reconstruction API - 15:50
```swift
var body: some View {
    ReconstructionProgressView()
        .task {
            var configuration = PhotogrammetrySession.Configuration()
            configuration.checkpointDirectory = getDocumentsDir()
                .appendingPathComponent("Snapshots/")
            let session = try PhotogrammetrySession(
                input: getDocumentsDir().appendingPathComponent("Images/"),
                configuration: configuration)
            try session.process(requests: [ 
                .modelFile(url: getDocumentsDir().appendingPathComponent("model.usdz")) 
            ])
            for try await output in session.outputs {
                switch output {
                    case .processingComplete:
                        handleComplete()
                        // Handle other Output messages here.
                }
            }
        }
}
```

[[Create 3D models with Object Capture]]

detail level on iOS.  We support only the reduced level on iOS.  Reconstructed model includes diffuse, AO, and normal.

IF you would like to generate other scale levels, transfer your images to a mac.  This year, mac reconstruction also uses lidar data.

There's a limit for how much on-device reconstruction can use.  U se `isOverCaptureEnabled=true` to indicate that we're not reconstructing on device so we don't need these limits.
### Capturing for Mac - 17:02
```swift
// Capturing for Mac

var configuration = ObjectCaptureSession.Configuration()
configuration.isOverCaptureEnabled = true

session.start(imagesDirectory: getDocumentsDir().appendingPathComponent("Images/"),
              configuration: configuration)
```

[[Meet Reality Composer Pro]] - object capture is integrated in there!  Simply import images into the app, choose a detail level, get your model.
# Reconstruction enhancements

* Increased performance on Mac.
* Estimated processing time
* Pose output
* custom detail level.

Can now request a high-quality pose for each image.  Each pose includes estimated position and orientation of the image.  Add `.poses` to function call.
### Pose output - 18:40
```swift
// Pose output

try session.process(requests: [ 
    .poses 
    .modelFile(url: modelURL),
])
for try await output in session.outputs {
    switch output {
    case .poses(let poses):
        handlePoses(poses)
    case .processingComplete:
        handleComplete()
    }
}
```

New custom detail level for macOS - full control over reconstructed model.  Previously, you could choose from reduced, medium, full, and raw.  With custom, you can control mesh estimatiion, texture map resolution, format, and which texture maps should be included.

# Wrap up
* scan more objects
* end-to-end iOS experience
* New use cases

# Resources
* https://developer.apple.com/documentation/RealityKit/guided-capture-sample
* 