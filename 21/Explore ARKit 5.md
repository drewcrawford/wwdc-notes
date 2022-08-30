#arkit 

# Location anchors
Maps is adding turn-by-turn walking directions
How location anchors can be used?
1.  Available on devices with A12 or newer and cellular (GPS) support
2.  Verify availability at location before use
3.  Camera and location permissions must be enabled

[[Explore ARKit 4]]
"Tracking Geographic Locations in AR" sample project

```swift
// Check device support for geo-tracking
guard ARGeoTrackingConfiguration.isSupported else {
    // Geo-tracking not supported on this device
    return
}

// Check current location is supported for geo-tracking
ARGeoTrackingConfiguration.checkAvailability { (available, error) in
    guard available else {
        // Geo-tracking is not available at this location
        return
    }

    // Run ARSession
    let arView = ARView()
    arView.session.run(ARGeoTrackingConfiguration())
}
```

Geo anchors can be added to ARSession like other types of anchors.  

```swift
// Create Location Anchor and add to session
let coordinate = CLLocationCoordinate2D(latitude: 37.795313, longitude: -122.393792)
let geoAnchor = ARGeoAnchor(name: “Ferry Building”, coordinate: coordinate)
arView.session.add(anchor: geoAnchor)

// Monitor geo-tracking status updates
func session(_ session: ARSession, didChange geoTrackingStatus: ARGeoTrackingStatus) {
    …
}
```

Important to monitor tracking status to see if you're localized.

## Supported locations
* 5 initially
* 25 cities (including Austin)
* London
* We'll be adding new regions over time
* Coordinate replay for development

## Onboarding with coaching overlay
New `.geoTracking` goal for coaching overlay
Animated guide to help achieve best experience
Familiarity from use across applications, including maps
Continue to monitor ARGeoTrackingStatus updates

```swift
// Declare coaching view
let coachingOverlay = ARCoachingOverlayView()

// Set up coaching view (assuming ARView already exists)
coachingOverlay.session = self.arView.session
coachingOverlay.delegate = self
coachingOverlay.goal = .geoTracking
     
coachingOverlay.translatesAutoresizingMaskIntoConstraints = false
self.arView.addSubview(coachingOverlay)

NSLayoutConstraint.activate([
    coachingOverlay.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    coachingOverlay.centerYAnchor.constraint(equalTo: view.centerYAnchor),
    coachingOverlay.widthAnchor.constraint(equalTo: view.widthAnchor),
    coachingOverlay.heightAnchor.constraint(equalTo: view.heightAnchor),
])
```

## Best practices
* Use #realitycomposer  to record ARKit sessions on devices
* Recordings can be shared with collaborators in remote locations
* Replay via Xcode
	* Use same device and iOS version
* Available for other ARKit features

1.  Open realitycomposer and tap upper right
2.  Open developer pane and Record AR Session

### Placing content
Consider creating content that floats in the air rather than trying to closely overlap real-world objects


* Obtaining lat/lon from Maps with 6 digits of precision
* Adjust altitude of content in app relative to anchor if needed
* Run application within 50 meters of anchor to get precise altitude
* Check anchor altitude source is `.precise`

```swift
// Method to compute distance (in meters) between points
func distance(from location: CLLocation) -> CLLocationDistance
```


# App Clip Codes
Small slice of app which takes peopel through one contextual workflow without installing whole app.

All codes contain visual pattern, and some have NFC tags.  

Preview swatch on wall, withotu downloading app.  

* Detect and track App Clip Codes in your AR App Clips and Apps
* Works on iOS devices with 14.3+ and Apple Neural Engine
* ARAppClipCodeAnchor
	* url
	* urlDecodingState
		* Can take longer to decode URL, depending on user's distance and other factors
		* decoding => still decoding
		* decoded => completed decoding
		* failed => not possible.  
			* > may be an app clip code not associated with the app.
	* radius
```swift
/// Accessing the URL of an App Clip Code 
override func session(_ session: ARSession, didUpdate anchors: [ARAnchor]) {
    for anchor in anchors {
        guard let appClipCodeAnchor = anchor as? ARAppClipCodeAnchor, appClipCodeAnchor.isTracked else { return }
        
        switch appClipCodeAnchor.urlDecodingState {
        case .decoding:
            displayPlaceholderVisualizationOnTopOf(anchor: appClipCodeAnchor)
        case .failed:
            displayNoURLErrorMessageOnTopOf(anchor: appClipCodeAnchor)
        case .decoded:
            let url = appClipCodeAnchor.url!
            let anchorEntity = AnchorEntity(anchor: appClipCodeAnchor)
            arView.scene.addAnchor(anchorEntity)
            let visualization = AppClipCodeVisualization(url: url, radius: appClipCodeAnchor.radius)
            anchorEntity.addChild(visualization)
          }
    }
}
```

Instant feedback that it was detected.

Consider positioning AR content relative to the app clip code

Combine app clip code detection with other tracking technolodies
"Interacting with App Clip Codes in AR" sample code

App Clips are supposed to contain only one workflow.

```swift
/// Adding a gesture recognizer for user interaction
func viewDidLoad() {
    initializeARView()
    initializeCoachingOverlays()
        
    // Place sunflower on the ground when the user taps the screen
    let tapGestureRecognizer = UITapGestureRecognizer(
     target: self,
     action: #selector(handleTap(recognizer:)))
    arView.addGestureRecognizer(tapGestureRecognizer)
}
```

Cast the ray into the world and get back a resulting location on the horizontal plane.  

Best practices for app clip codes:

* Consider whether NFC or visual codes are more appropriate for your use case
* Add call to action that informs people how to use the App Clip Code
* For visual codes, use a minimum of diameter size of at least 1 inch / 2.5 cm

| Scenario        | Good App Clip Code diameter | Detected from a maximum distance |
|-----------------|-----------------------------|----------------------------------|
| Restaurant menu | 1 in (2.5cm)                | 1 ft 8 in (50cm)                 |
| Movie poster    | 5 in (12.5 cm)              | 8 ft 4 in (250cm)                |

[[21/What's new in App Clips]]
[[Build light and fast App Clips]]

# Face tracking
Last year we expanded to A12+
Ultrawide iPad pro.  

Be aware that existing apps will continue using the normal camera.  Check which video formats are available 

```swift
// Check if the ultra wide video format is available.
// If so, set it on a face tracking configuration & run the session with that.

let config = ARFaceTrackingConfiguration()
for videoFormat in ARFaceTrackingConfiguration.supportedVideoFormats {
    if videoFormat.captureDeviceType == .builtInUltraWideCamera {
        config.videoFormat = videoFormat
        break
    }
}
session.run(config)
```

## Face tracking
* capturedDepthData is not available on ARFrame when using the ultra wide camera

# Motion capture
2D/3D, etc.  In iOS 15, motion capture is getting better.  On A14+, now supports
* wider range of body postures
* No change required in your app

Track body joints from further distance
limb movement range

# Wrap up
* Location anchors
* App Clip Codes
* Face tracking
* Motion capture

* https://developer.apple.com/design/human-interface-guidelines/app-clips/overview/app-clip-codes
* https://developer.apple.com/documentation/app_clips/interacting_with_app_clip_codes_in_ar
* https://developer.apple.com/forums/tags/arkit
* https://developer.apple.com/documentation/arkit/content_anchors/tracking_geographic_locations_in_ar
* https://developer.apple.com/documentation/arkit



