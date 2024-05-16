Speed up feature development by prototyping new code with Xcode Playgrounds, eliminating the need to keep rebuilding and relaunching your project to verify your changes. We'll show you how using a playground in your project or package can help you try out your code in various scenarios and take a close look at the returned values, including complex structures and user interface elements, so you can quickly iterate on a feature before integrating it into your project.

# Rapid iterations

### birdsToShow computed property before the required changes - 2:04
```swift
var birdsToShow: [Bird] {
    // TODO: Narrow down the list of birds to find.
    let birdProvider = BirdProvider(region: .northAmerica)
    return birdProvider.birds
}
```

Adding a new playground to my project.  

choose 'automatically run' in bottom bar.

'Build Active Scheme' and 'Import App Types'.  Ensure that the active scheme is built before app execution, and that app target modules are automatically import.  Work with types defined within your project!


Inline resutl shows details of BirdProvider instance along with two properties; birds array and region.
### birdProvider instance - 4:15
```swift
let birdProvider = BirdProvider(region: .northAmerica)
```

### CustomStringConvertible - 6:31
```swift
extension Bird: CustomStringConvertible {
    
    public var description: String {
        return "\(commonName) (\(scientificName))"
    }
    
}
```

### CustomPlaygroundDisplayConvertible - 8:17
```swift
extension Bird: CustomPlaygroundDisplayConvertible {
    
    public var playgroundDescription: Any {
        return photo as Any
    }
    
}
```


### Birds without a photo - 9:45
```swift
let birdsToFind = birdProvider.birds.filter { $0.photo == nil }
```
### Owls without a photo - 10:52
```swift
let owlsToFind = birdsToFind.filter { $0.family == .owls }
```

Note that we show all expressions in righthand panel, hover over to see their source locations.  For repeated expressions / ex filter we show latest result.

### Verifying the ChecklistView - 11:15
```swift
let checklist = ChecklistView()
for bird in owlsToFind {
    checklist.add(bird)
}
```

for more on string catalogs, see [[Discover String Catalogs]]


### birdsToShow computed property with the required changes - 13:41
```swift
var birdsToShow: [Bird] {
    let birdProvider = BirdProvider(region: .northAmerica)
    let birdsToFind = birdProvider.birds.filter { $0.photo == nil }
    let owlsToFind = birdsToFind.filter { $0.family == .owls }
    return owlsToFind
}
```


### sightingToShow function before the required changes - 14:21
```swift
func sightingToShow(for bird: Bird) -> Sighting? {
    // TODO: Use BirdSightings package to fetch the most recent sighting.
    return nil
}
```
### BirdSightings package example - 15:04
```swift
let barnOwlCode = "BNOW"
let centralParkLocation = CLLocationCoordinate2D(latitude:  40.785091, longitude: -73.968285)

let sightingsProvider = SightingsProvider()
sightingsProvider.fetchSightings(of: barnOwlCode, around: centralParkLocation)
```


### Parameters for sightings fetching - 16:08
```swift
let bird = owlsToFind.first!
let appleParkLocation = CLLocationCoordinate2D(latitude: 37.3348655, longitude: 122.0089409)
```
### Sightings fetching - 17:31
```swift
let sightingsProvider = SightingsProvider()
let sightings = sightingsProvider.fetchSightings(of: bird.speciesCode, around: appleParkLocation)
```


### Initializing the SightingMapView - 18:23
```swift
let mostRecentSighting = sightings.first
let sightingMapView = SightingMapView(sighting: mostRecentSighting)
```
### Setting up the playground's live view - 18:55
```swift
import PlaygroundSupport
PlaygroundPage.current.liveView = sightingMapView
```
### sightingToShow function with the required changes - 22:20
```swift
func sightingToShow(for bird: Bird) -> Sighting? {
    let sightingsProvider = SightingsProvider()
    let sightings = sightingsProvider.fetchSightings(of: bird.speciesCode, around: lastCurrentLocation)
    let mostRecentSighting = sightings.first
    return mostRecentSighting
}
```

# Exercise rare codepaths

# Try out new dependencies

# Wrap up
Customize playground representation
Take advantage of execution modes
Use value history mode
Work with live views