MapKit JS provides a JavaScript API to embed interactive Apple Maps directly into your webpages or apps across different platforms and operating systems, including iOS and Android. Learn about the latest features to help improve load performance and make your web and native apps more responsive and faster — all while giving you more control.

Best possible experience integrating mapkit JS.

Since introduction of mapkit JS in 2018, we've worked tirelessly adding new features each year.  MapKit JS has become an indispensable part of apple's webkit offering.

FindMy on icloud, ddg, etc.

# High performance loading markup

Can tune webapp with additional properties

* mapkit.core.js -> only minimum code to get started.
* data-libraries="map".  Here we specify just the map library, the minimum featureset to show an interactive map
* async -> don't block pageload
* data-callback to tell us when we've finished loading

If mapkit object is undefined, it hasn't been loaded yet.

check if initial libraries have been loaded as well `window.mapkit.loadedLibraries.length == 0`.

`crossorigin` attribute - reuse a single http connection for all requests going to our cdn.  We'd like mapkit to intiialize itself as soon as it has the opportunity to do so.  Set data-initial-token.

Since we only asked for the map library, we need to know the annotations library to know our pirate ship and treasures on the map.
# On-demand library loading
`mapkit.load("LIBRARY_NAME")`
Asynchronous method
dependencies and duplications handled automatically.

`mapkit.addEventListener("load", event => { addAnnotations()})`

* services (search, geocoder)
* full map (all features)
* map (basic map)
* overlays
* annotations
* user location
* GeoJSON


# Prioritized map start-up

* Customizable load priority over other map display properties
* Better perceived responsiveness
* Surface rich information sooner

* point of interest
* land cover
* none

# Wrap up
* new features designed for performance
* integration while maintaining load performance
* progressive improvements

[[Meet Apple Maps Server APIs]]


# Resources
* https://developer.apple.com/maps/web/
