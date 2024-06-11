Find out how you can use new enterprise APIs for visionOS to create spatial experiences that enhance employee and customer productivity on Apple Vision Pro.

###  Main Camera Feed Access Example - 3:36
```swift
let formats = CameraVideoFormat.supportedVideoFormats(for: .main)
let cameraFrameProvider = CameraFrameProvider(formats: formats)
var arKitSession = ARKitSession()
var pixelBuffer: CVPixelBuffer?

await arKitSession.queryAuthorization(for: [.cameraAccess])

do {
    try await arKitSession.run([cameraFrameProvider])
} catch {
    return
}

guard let cameraFrameUpdates = cameraFrameProvider.cameraFrameUpdates(for: formats[0]) else {
    return
}

for await cameraFrame in cameraFrameUpdates {
    guard let mainCameraSample = cameraFrame.sample(for: .left) else {
        continue
    }
    self.pixelBuffer = mainCameraSample.pixelBuffer
}
```

###  Spatial barcode & QR code scanning example - 7:47
```swift
await arkitSession.queryAuthorization(for: [.worldSensing])

let barcodeDetection = BarcodeDetectionProvider(symbologies: [.code39, .qr, .upce])

do {
    try await arkitSession.run([barcodeDetection])
} catch {
    return
}

for await anchorUpdate in barcodeDetection.anchorUpdates {
    switch anchorUpdate.event {
    case .added:
        // Call our app method to add a box around a new barcode
        addEntity(for: anchorUpdate.anchor)
    case .updated:
        // Call our app method to move a barcode's box
        updateEntity(for: anchorUpdate.anchor)
    case .removed:
        // Call our app method to remove a barcode's box
        removeEntity(for: anchorUpdate.anchor)
    }
}
```

###  Apple Neural Engine access example - 13:17
```swift
let availableComputeDevices = MLModel.availableComputeDevices

for computeDevice in availableComputeDevices {
    switch computeDevice {
    case .cpu:
        setCpuEnabledForML(true) // Example method name
    case .gpu:
        setGpuEnabledForML(true) // Example method name
    case .neuralEngine:
        runMyMLModelWithNeuralEngineAvailable() // Example method name
    default:
        continue
    }
}
```

###  Object tracking enhancements example - 15:39
```swift
var trackingParameters = ObjectTrackingProvider.TrackingConfiguration()

// Increasing our maximum items tracked from 10 to 15
trackingParameters.maximumTrackableInstances = 15

// Leaving all our other parameters at their defaults
trackingParameters.maximumInstancesPerReferenceObject = 1
trackingParameters.detectionRate = 2.0
trackingParameters.stationaryObjectTrackingRate = 5.0
trackingParameters.movingObjectTrackingRate = 5.0

let objectTracking = ObjectTrackingProvider(
    referenceObjects: Array(referenceObjectDictionary.values),
    trackingConfiguration: trackingParameters
)

var arkitSession = ARKitSession()
arkitSession.run([objectTracking])
```

# Resources
https://developer.apple.com/documentation/visionOS/building-spatial-experiences-for-business-apps-with-enterprise-apis
