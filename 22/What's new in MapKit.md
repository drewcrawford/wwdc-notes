All new maps and lookaround coverage has been expanding since.

Last year we introduced 3d city experiences with turn lanes, crosswalk, bike lanes, trees,e tc.

Provide context and precision that was never befoere possible.  Terrian elevation provides a level of realism like no other maps.

In this talk, we'll be covering several new MapKit features.  First, 

# Map Configuration
Simply recompile your app with the new SDK and it will be automatically opted in, including 3d experience where available.  For many apps, recompile is all you need.

Maybey ou nee more control over presentation.  In iOS 15, there are various properties on MKMapView.  We're soft-deprecating these, and replacing MKMapConfiguration API.

Central class of new configuration API.  3 subclasses.  
* Imagery => Used to prevent stylized style imagery
* MKHybridMapConfiguration => Road labels, points of interest, etc.
* MKSTandardMapConfiguration => fully graphic space map.  Similar to existing map types

MKMapConfiguration => elevationStyle can be either flat or realistic.  Flat style means that the ground appears flat.  roads, bridges, overpasses, appear flat.  Default.

realistic => ground terrain reproduces the real world elevation.  Roads depicted with realistic elevation details.

* Imagery
	* Only shows satellite imagery
* HybridMapConfiguration
	* pointsOfInterestFilter
	* showTraffic
* Standard
	* emphasisStyle => default or muted.  softens contrast of map details allowing you to bring more attention to your information on top.
	* showsTraffic, pointOfInterestFilter, etc.

Only configure supported options.  Configure map atomically.

## Device support
3D experience:
A12 iPhone and iPad
M1 Mac

Map automatically falls back to all-new map:
* watch
* older iPhone and iPad
* Intel Mac

Simulator:
* m1 mac
* both experiences "by changing the OS version"

SF bay, LA, NYC, London, WAshington DC, san diego, philadelphia, etc.


# Overlay improvements
Two different levels.
* aboveLabels
	* above everything, including labels.
	* Only use in rare cases where you don't want your data to interact with the map at all.
	* If you want your content to stand out, consider using muted map emphasis or blend modes
* aboveRoads
	* Shown on top of terrain, bodies of water, roads, etc.  Shown below labels, and to some degrees, trees and buildings
	* New default mode in iOS 16

Regardless of whetehr your overlay is aboveRoads or aboveLabels, will always be rendered above buildings with top-down and no pitch.

Now ground objects such as trees and buildings are automatically rendered with transparency on overlays.  Alpha value varies with pitch angle.  0 degree pitch, objects disappear from view.  

Also work for semi-transparent overlays.  We add the alpha values.

When adding an overlay to a map with realistic train, mapkit will transition to flat representation.  Map will go back to realistic when you remove the last overlay.  Exception: overlay sourced through mapkit's direction API.  Those overlays follow terrain.

## Demo
Configure mapview in IB.

Polyline returned by directions API will retain the realistic elevation.
elevation is supported on A12+ devices.  On older devices i get 2d route on 2d map.
# Blend modes
Control overlay look and feel.  photo editing apps and graphics APIs.  During a blend operation, graphical layers are combined following a set of equations.

In this scenario, I want to highlight area.  Color burn, etc.

Set desired CGBlendMode. on `MKOverlayRenderer`.

`insertOverlay(_ overlay: atIndex: Int, level: MKOverlayLevel)`.

MapKit supports a wide range of blend modes.

# Selectable Map Features
Unless you're using POI filterign, you''re placing those annotations on a map that uses another of annotation.  Up until now, your users could only interact with annotations you provided.

Selectable map features gives the option to let your users select features on the map.
* Points of interest
* Territories
	* cities, states, countries
* Physical features
	* mountain rangers, lakes

1.  Configure which features should be selectable
	2. `.selectableMapFeatures`
3. Implement selection delegate callbacks
	4. `didSelect`, `didDeselect`, etc.
	5. `mapView(viewFor annotation:)`
		6. return `nil` to get maps-like behavior
5. Request and dispaly additional place information
	6. `MKmapItemRequest(featureAnnotation:)`
	7. can obtain map item for the requested feature.
	8. address, name, phone number, URL, etc.
	9. `openInMaps`.

## Demo
Can configure filters in IB.

`selectedGlyphImage` vs `glyphImage`
`.iconStyle` has iconography and color info of selected POI.

# Look Around
Available in many placess around the world, and entire countries.  Check out the section on the feature availability website.

With iOS 16, we're bringing this to mapkit.  Adopting it only requires 3 steps.
* check for data availability
	* not every location can be seen from a street
* Pass data to the Look Around UI
* Conditionally show Look Around UI

`MKLookAroundSceneRequest`.  `.init(coordinate:CLlocationCoordinate2D)` or map item.
`.scene`.  If data is available, you will get an instance.  If not available, get `nil`.
`MKLookAroundScene` => opaque object that acts as a token.

Pass to `MKLookAroundViewController`.  
Or `MKLookAroundSnapshotter`.  

Make ita s easy as possible to embed a smaller static preview.  The user can tap on to enter a full-screen interactive session.

## Demo
When users tap on one segmented control, we want to perform a camera animation, etc.  

LookaroundVC in IB.

# Wrap up
* Share your feedback
* Check out updated samples
* Maps Server APIs

[[Meet Apple Maps Server APIs]]


* https://www.apple.com/ios/feature-availability/
* https://developer.apple.com/documentation/mapkit/explore_a_location_with_a_highly_detailed_map_and_look_around
* https://developer.apple.com/documentation/mapkit/interacting_with_nearby_points_of_interest
* https://developer.apple.com/maps/mapkitjs/
* https://developer.apple.com/maps/
* https://developer.apple.com/documentation/mapkit