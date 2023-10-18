#swiftui #mapkit #maps 
Discover how expanded SwiftUI support for MapKit has made it easier than ever for you to integrate Maps into your app. We'll show you how to use SwiftUI to add annotations and overlays to a map, control the camera, and more.

Fully-functional trip planner from scratch.

# Show content on the map
* mark the parking garage

map automatically zooms in to show marker.  Just puass them into content builder.

Can present overlapy content.  polyline, circle, etc.

# Give the user a sense of place.
Enable realistic terrain elevation.
Display satellite and flyover imagery

Imagery - satellite, etc.
hybrid - imagery with labels.

# Search for places
* display buttons above the map
* show search-result markers

Create markers automatically from mapitems - get stuff populated correctly, etc.

can provide your own icon if created manually.  Show up to 3 letters of text with monograms.

# display a place or region
* control what place is displayed
* discover if the user has interacted

I need to reset the map's camera position state, after the user has interacted with the map.
use `@State` to track position.
default `.automatic` position.  Use `Map(position:$position)


Explicit
* MapCamera
Semantic position:
* MapCameraPosition

mapkit takes care of the camera for me.  
* `automatic`
* `region(MKCoordianteRegion)` to show specific region
* `rect(MKMapRect)`.  Show an area just like how we'd use region.
* `item(MKMapItem)
* camera - configure exactly how you want, pitch angle, etc.
* userLocation - fallback when location is not known

MapKit writes to the binding when position changes.  ex if the user pans away and not following the user, we write to the binding?

discover if the user has interacted `positionedByUser`.

# Search in the visible region
* respond when the MapCamera changes
* by default, closure `onMapCameraChange` at end.  Can also request continuous updates.

visibleMapRect, and also map camera itself.

# Interact with search results
* enable selection for map content

map needs selection binding, in constructor.  

If you want to support selection, you can simply tag items.  This works the same way for picker/list.  Here, teh selected tag state is an int.  So the binding eanbles selection for both of them.  When using tag, use any type conforming to Hashable.

# Display useful search-result information

look around
place name
travel time


# Show driving route using MapPolyLine
Other overlay types
* MapPolygon
* map Circle

aboveRoads vs aboveLabels

# User location and map controls
* show the user's location
* Learn about Map Control views

`UserAnnotation()` in builder block.

`.mapControls` builder.
By default, we show a compass while rotated, and a scale when zooming.  

Kind of an interesting design here to have an additional builder.

Can present them in your own UI.  Simply create a `Map` with a `scope`, and then use the scope to bind to the controls.

map style, lookaround, etc.  Give your users a real sense of place.

# Wrap up

maps server APIs
* autocomplete
* directions

[[Meet Apple Maps Server APIs]]

[[SwiftUI Animation Plans]]




# Resources
* https://developer.apple.com/documentation/mapkit
* https://developer.apple.com/documentation/mapkit/mapkit_for_swiftui
* 