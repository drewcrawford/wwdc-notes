Discover powerful new ways to integrate maps into your apps and websites with MapKit and MapKit JS. Learn how to save and reference unique places using Place ID. Check out improvements to search that make it more efficient to find relevant places. Get introduced to the new Place Card API that lets you display rich information about places so customers can explore destinations right in your app. And, we'll show you quick ways to embed maps in your website with our simplified token provisioning and Web Embed API.

This year, we've added new features to bring places into your own apps in new ways.

3 primary usecases.

# Reference a place
Place ID `MKMapItme.identifier` or `Place.id`.
References a business, landmark, and more.
References are unique, remain valid over time.  Last as long as places do.
Always provides our latest data.
Extend your business data

What better starter than the apple park visitor center?  Give the place ID for the center.  Could use an existing API like search, geocoding.  Instead, new placeID lookup tool.

developer.apple.com/maps
Resources
Place ID lookup

Display apple park visitor center.  Create identifier with placeID from lookup tool.  Fetch map item using async request.  App simply displays a map with a marker containing that item.  Instead of using hardcoded coordinate, leveraging strength of apple maps editor.


###  3:06 - Display a visitor center annotation
```swift
// Display a visitor center annotation 
struct PlaceMapView: View { 
    var placeID: String // "I63802885C8189B2B" 
    @State private var item: MKMapItem? 
    
    var body: some View { 
        Map { 
            if let item { 
                Marker(item: item) 
            } 
        } 
        .task { 
            guard let identifier = MKMapItem.Identifier( rawValue: placeID ) else { 
                return 
            } 
            let request = MKMapItemRequest( mapItemIdentifier: identifier ) 
            item = try? await request.mapItem 
        } 
    } 
}
```

JS example


###  3:44 - Display an annotation for the center
```html
<!DOCTYPE html> 
<html> 
<head> 
<meta name="viewport" content="width=device-width, initial-scale=1" /> 
<style> 
#map { margin: 0 auto; } 
</style> 
</head> 
<body> 
<script crossorigin async src="https://cdn.apple-mapkit.com/mk/5.x.x/mapkit.js" data-callback="entryPoint" data-token="TODO: Add your token here" ></script> 
<script> 
window.entryPoint = () => { 
    const id = "I63802885C8189B2B"; 
    const lookup = new mapkit.PlaceLookup(); 
    lookup.getPlace(id, annotatePlace); 
}; 

const annotatePlace = (error, place) => { 
    const center = place.coordinate; 
    const span = new mapkit.CoordinateSpan(0.01, 0.01); 
    const region = new mapkit.CoordinateRegion(center, span); 
    const map = new mapkit.Map("map", { region }); 
    const annotation = new mapkit.PlaceAnnotation(place); 
    map.addAnnotation(annotation); 
}; 
</script> 

<div id="map" style="width: 100dvw; height: 100dvh;"></div> 
</body> 
</html>
```


Less code than ever to initialize MapKit JS.  Just add this async script tag with the MapKit JS src attribute and list my entryPoint function as the data callback.

This year, we're replacing dynamic JWT with a domain-specific token strings that don't have to expire.  

Resources page, 'Create a token' tool.  

Copy my token right into script tag.  No worries about expiration dates, JWT expiration, etc.

'Identifying unique locations with placeIDs' in docs.
# Display place details

When you tap on a place, it displays the place card.  Opening hours, phone number, and more.  This year, MapKit brings place cards to your apps and websites.  Our SelectionAccessory APIs are used to display info.

* Full
* Compact
* Link


###  7:32 - Display my favorite apple stores
```swift
// Display my favorite apple stores 
struct VisitedStoresView: View { 
    var visitedStores: [MKMapItem] 
    @State private var selection: MKMapItem? 
    
    var body: some View { 
        Map(selection: $selection) { 
            ForEach(visitedStores, id: \.self) { store in 
                Marker(item: store) 
            } 
            .mapItemDetailSelectionAccessory() 
        } 
    } 
}
```

Present even if your app/website doesn't feature a mapview.  Map item detail and place detail APIs add flexibility.  Embeds are a quick way to add a map to your website.


###  7:50 - Display a selectable annotation
```html
<!DOCTYPE html> 
<html> 
<head> 
<meta name="viewport" content="width=device-width, initial-scale=1" /> 
<style> 
#map { margin: 0 auto; } 
</style> 
</head> 
<body> 
<script crossorigin async src="https://cdn.apple-mapkit.com/mk/5.x.x/mapkit.js" data-callback="entryPoint" data-token="TODO: Add your token here" ></script> 
<script> 
window.entryPoint = () => { 
    const id = "I63802885C8189B2B"; 
    const lookup = new mapkit.PlaceLookup(); 
    lookup.getPlace(id, annotatePlace); 
}; 

const annotatePlace = (error, place) => { 
    const center = place.coordinate; 
    const span = new mapkit.CoordinateSpan(0.01, 0.01); 
    const region = new mapkit.CoordinateRegion(center, span); 
    const map = new mapkit.Map("map", { region }); 
    const annotation = new mapkit.PlaceAnnotation(place); 
    map.addAnnotation(annotation); 
    const accessory = new mapkit.PlaceSelectionAccessory(); 
    annotation.selectionAccessory = accessory; 
}; 
</script> 

<div id="map" style="width: 100dvw; height: 100dvh;"></div> 
</body> 
</html>
```

Sometimes I want a quick list item without a map.  This shows a full placecard when tapped.

###  9:15 - List stores and show details when selected
```swift
// List stores and show details when selected 
struct StoreList: View { 
    var stores: [MKMapItem] 
    @State private var selectedStore: MKMapItem? 
    
    var body: some View { 
        List( 
            stores, 
            id: \.self, 
            selection: $selectedStore 
        ) { 
            Text($0.name ?? "Apple Store") 
        } 
        .mapItemDetailSheet(item: $selectedStore) 
    } 
}
```


Match system's preferred color scheme - light or dark.
###  9:37 - Show visitor center details
```html
<!DOCTYPE html> 
<html> 
<head> 
<meta name="viewport" content="width=device-width, initial-scale=1" /> 
<style> 
#map { margin: 0 auto; } 
</style> 
</head> 
<body> 
<script crossorigin async src="https://cdn.apple-mapkit.com/mk/5.x.x/mapkit.js" data-callback="entryPoint" data-token="TODO: Add your token here" ></script> 
<script> 
window.entryPoint = () => { 
    const id = "I63802885C8189B2B"; 
    const lookup = new mapkit.PlaceLookup(); 
    lookup.getPlace(id, annotatePlace); 
}; 

const annotatePlace = (error, place) => { 
    const el = document.getElementById("place"); 
    const detail = new mapkit.PlaceDetail(el, place, { 
        colorScheme: mapkit.PlaceDetail.ColorSchemes.Adaptive 
    }); 
}; 
</script> 

<div id="place"></div> 
</body> 
</html>
```

| language | api                                        |
| -------- | ------------------------------------------ |
| Swift    | MKMapItemDetailViewController              |
| SwiftUI  | mapItemDetailSheet<br>mapItemDetailPopover |
| JS       | PlaceDetail                                |
selection accessory api

| language | API                             |
| -------- | ------------------------------- |
| Swift    | MKSelectionAccessory            |
| SwiftUI  | mapItemDetailSelectionAccessory |
| JS       | PlaceSelectionAccessory         |
| HTML     | Web Embeds                      |
Enable placecard display for all the other places on a map, not just the apple stores.  


###  11:17 - Display a place card for the selected map feature, too
```swift
// Display a place card for the selected map feature, too 
struct VisitedStoresView: View { 
    var visitedStores: [MKMapItem] 
    @State private var selection: MapSelection<MKMapItem>? 
    
    var body: some View { 
        Map(selection: $selection) { 
            ForEach(visitedStores, id: \.self) { store in 
                Marker(item: store) 
                    .tag(MapSelection(store)) 
            } 
            .mapItemDetailSelectionAccessory(.callout) 
        } 
        .mapFeatureSelectionAccessory(.callout) 
    } 
}
```

| language | api                                                             |
| -------- | --------------------------------------------------------------- |
| Swift    | selectableMapFeatures<br>MKSelectionAccessory                   |
| SwiftUI  | MapFeature<br>MapSelection<br>mapFeatureSelectionAccessory      |
| JS       | selectableMapFeatures<br>selectableMapFeatureSelectionAccessory |
| HTML     | Web Embeds                                                      |

# Find a place
Search filters
* place categories
* physical features
* address components
Region priority
Pagination (Maps Server API)

"Locality" is a city in most countries.
###  13:09 - Find Cupertino, then find coffee
```html
<!DOCTYPE html> 
<html> 
<head> 
<meta name="viewport" content="width=device-width, initial-scale=1" /> 
<style> 
#map { margin: 0 auto; } 
</style> 
</head> 
<body> 
<script crossorigin async src="https://cdn.apple-mapkit.com/mk/5.x.x/mapkit.js" data-callback="entryPoint" data-token="TODO: Add your token here" ></script> 
<script> 
window.entryPoint = () => { 
    const addressFilter = mapkit.AddressFilter.including([ 
        mapkit.AddressCategory.Locality 
    ]); 
    
    const citySearch = new mapkit.Search({ 
        addressFilter 
    }); 
    
    citySearch.search("Cupertino", showMap); 
}; 

const showMap = (error, cities) => { 
    const center = cities.places[0].coordinate; 
    const span = new mapkit.CoordinateSpan(0.01, 0.01); 
    const region = new mapkit.CoordinateRegion(center, span); 
    const map = new mapkit.Map("map", { region }); 
    
    const coffeeSearch = new mapkit.Search({ 
        region, 
        regionPriority: mapkit.Search.RegionPriority.Required, 
        pointOfInterestFilter: mapkit.PointOfInterestFilter.including([ 
            mapkit.PointOfInterestCategory.Cafe 
        ]) 
    }); 
    
    coffeeSearch.search("coffee", (error, results) => { 
        for (const place of results.places) { 
            const marker = new mapkit.PlaceAnnotation(place); 
            map.addAnnotation(marker); 
        } 
    }); 
}; 
</script> 

<div id="map" style="width: 100dvw; height: 100dvh;"></div> 
</body> 
</html>
```

RegionPriority to remove unneeded places.  Results are within the region I provided within the search.  Results are limited to just what's in the original region.

Available to native apps as well.

###  14:41 - Finding coffee in Cupertino
```swift
// Finding coffee in Cupertino 
struct CoffeeMap: View { 
    @State private var position: MapCameraPosition = .automatic 
    @State private var coffeeShops: [MKMapItem] = [] 
    
    var body: some View { 
        Map(position: $position) { 
            ForEach(coffeeShops, id: \.self) { café in 
                Marker(item: cafe) 
            } 
        } 
        .task { 
            guard let cupertino = await findCity() else { 
                return 
            } 
            coffeeShops = await findCoffee(in: cupertino) 
        } 
    } 
    
    private func findCity() async -> MKMapItem? { 
        let request = MKLocalSearch.Request() 
        request.naturalLanguageQuery = "cupertino" 
        request.addressFilter = MKAddressFilter( including: .locality ) 
        let search = MKLocalSearch(request: request) 
        let response = try? await search.start() 
        return response?.mapItems.first 
    } 
    
    private func findCoffee(in city: MKMapItem) async -> [MKMapItem] { 
        let request = MKLocalSearch.Request() 
        request.naturalLanguageQuery = "coffee" 
        let downtown = MKCoordinateRegion( 
            center: city.placemark.coordinate, 
            span: .init( 
                latitudeDelta: 0.01, 
                longitudeDelta: 0.01 
            ) 
        ) 
        request.region = downtown 
        request.regionPriority = .required 
        let search = MKLocalSearch(request: request) 
        let response = try? await search.start() 
        return response?.mapItems ?? [] 
    } 
}
```

Where we take the cupertino region and use a region priority to search for coffee shops within the region.

Well on my way to be the very best apple store collector.

Server API offers more results than ever with pagination.  Simplified token provisioning process, etc.

# Wrap up
* Identify Place IDs and store them
* Dispaly rich information with place card apis
* Improve your search experience



# Resources
* https://developer.apple.com/documentation/mapkitjs/displaying_place_information_using_the_maps_embed_api
* https://developer.apple.com/documentation/mapkit/mapkit_for_appkit_and_uikit/identifying_unique_locations_with_place_ids
* https://developer.apple.com/documentation/mapkit/mapkit_for_appkit_and_uikit/interacting_with_nearby_points_of_interest
* https://developer.apple.com/maps/resources/
