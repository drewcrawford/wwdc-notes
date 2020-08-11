#arkit 

# ARKit 4
* Location anchors
* scene geometry
* depth
* Object placement
* Face tracking

# Location anchors
Place AR content in relation to the globe.
Geo-referenced AR content
Apple maps visual localization
`ARGeoTrackingConfiguration`
`ARGeoAnchor`

We download "localization data", a.k.a. depth points, from apple maps.

* `ARGeoTrackingConfiguration` -> take advantage of new location features.  This contains a *subset* of the features, that are compatible with geo tracking.
* `ARGeoAnchor`
* `ARGeoTrackingStatus` -> provides valueable feedback

# Steps
* check availability
* adding location anchors
* geo-tracking state transitions

## availability
* Location anchors are available on a12+ with GPS
* Not available in all locations, because sometimes maps data is not available.  Therefore we need to check user location as well.
* ARKit will ask the user for camera and location.  

```swift
guard ARGEoTrackingConfiguration.isSupported else { return }
ARGeoTrackingConfiguration.checkAvailability { (available, error ) in 
	guard available else {
		//not available at this location
		return
	}
	let arView = ARView()
	arView.session.run(ARGEoTrackingConfiguration())
}
```

It's important to use 6 or more digits of latitude/longitude to get precise data.

```swift
let coordinate = CLLocationCoordinate2D(latitude: , longitdue)
...
```

## positioning content
`ARGeoAnchors` are fixed to cardinal directions.

X axis is pointed east, Z axis is pointed south.  This leaves +y pointing up.
Use rendering engine to offset virtual content.

## Geo-tracking state transitions

Similar to world-tracking information.  `state` indicates how far along geotracking is.  Also a reason.  Also accuracy.

1.  Initializing
2.  Not available (if not supported in this location)
3.  1 -> Localizing
4.  localized

`ARGeoTrackingStateReason`
* `devicePointedTooLow` -> inform user to raise the device
* `geoDataNotLoaded` -> network connection is required

Point your device at buildings visible from the street.  Stuff that dynamically changes is not a good option.

## Authoring location anchors
Another way to create a location anchor is via user interaction.  

# Scene geometry
* topological map of the environment
* semantic classification
* occlusion
* physics
* virtual lighting

## Depth
Provides dense depth image -> pixel is m from camera
Depth map with each ARFrame -> 60hz
Foundation of scene geometry API

## Depth map
`CVPixelBuffer` where each pixel represents depth in meters
distance from camera plane to points in the world
smaller resolution compared ot captured image

## Confidence map
Accuracy of depth impacted by environment.
Challenging surfaces such as highly reflective or with high absorption may lower accuracy
Confidence value for each depth value, `ARConfidenceLevel`

# Improvements in raycasting
Optimized for precise object placement
Quicker results with LiDAR
Benefits from scene depth and scene geometry
Recommended over hit-testing

## Raycast query
* Target – existing planes, infinite planes, or estimated planes
* Target alignment – horiziontal, vertical, or any

Raycast – oneshot
Tracked raycast – continually update

```swift
let query = arView.makeraycastQuery(from: point, allowing: .estimatedPlane, alignment: .any)!

let raycast = arView.session.trackedRaycast(query) {results in
}
```

# Face tracking
Extended to devices without the trudepth camera, as long as they have a12+