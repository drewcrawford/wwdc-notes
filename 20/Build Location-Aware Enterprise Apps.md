# Build Location-Aware Enterprise Apps
# About Caffe Macs
In-house enterprise apps where you can brows the menu, etc.
In-house kitchen display system



# Defining user preferences
Allow users to select their own default location
* Closest to me
* A specific Caffe

These options give users control over which menu they see.

```swift
// Storing the user’s preference using UserDefaults

UserDefaults.standard.set(defaultLocation.id, forKey: "defaultLocationId")

let defaultLocationId = UserDefaults.standard.integer(forKey: "defaultLocationId")
```


# Using location services
An authorization status determines if and when your app receives location events.

Employees choose whether or not to grant permission to enterprise apps to location services.  Explain to the user why you need it.

[[Design for Location Privacy]]

Your app can request
* When in use
* Always

For more about requesting authorization, including the approximate request, wee [[What’s New in Core Location]]
Note that even if you ask for authorization, you may not be guaranteed to receive it.  Handle failure gracefully.
Configure purpose strings in Xcode.

```swift
// Add NSLocationWhenInUseUsageDescription to your Info.plist 
// e.g. “Location is required for placing orders while using the app."
//otherwise this will fail without warning
locationManager.requestWhenInUseAuthorization()

func locationManager(
    _ manager: CLLocationManager,
    didChangeAuthorization status: CLAuthorizationStatus) {
        
    switch status {
    case .restricted, .denied: 
        disableLocationFeatures()

    case .authorizedWhenInUse, .authorizedAlways: 
        enableLocationFeatures()

    case .notDetermined: // The user hasn’t chosen an authorization status
    }
}
```

`CLLocationManager` provides methods for determining device support.  Not all devices are supported.  Can fall back on the default user preference that we discussed earlier.

```swift
if CLLocationManager.isMonitoringAvailable(for: CLBeaconRegion.self) {
    // Supports region monitoring to detect beacon regions
}

if CLLocationManager.isRangingAvailable() {
    // Supports obtaining the relative distance to a nearby iBeacon device
}
```

A beacon is a device that emits a signal that can be detected by the system.

Set UUID, major value and minor value.
Allow apps to use beacon proximity to determine a course of action
Use region monitoring to detect the presence of a beacon
Use ranging to determine the proximity of a detected beacon

[[What’s New in Core Location - 19]]

```swift
// Stage 1: Region Monitoring

func monitorBeacons() {
    if CLLocationManager.isMonitoringAvailable(for: CLBeaconRegion.self) {

        let constraint = CLBeaconIdentityConstraint(uuid: proximityUUID)

        let beaconRegion = CLBeaconRegion(
            beaconIdentityConstraint: constraint,
            identifier: beaconID //only beacons with matching values
        )
        
        self.locationManager.startMonitoring(for: beaconRegion)
    }
}
```

```swift
// Stage 2: Beacon Ranging

func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
    guard let region = region as? CLBeaconRegion,
        CLLocationManager.isRangingAvailable()
        else { return }
    
    let constraint = CLBeaconIdentityConstraint(uuid: region.uuid)
    manager.startRangingBeacons(satisfying: constraint)
    beaconsToRange.append(region)
}

func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
    
}
```

```swift
// Stage 2: Beacon Ranging

func locationManager(
    _ manager: CLLocationManager,
    didRangeBeacons beacons: [CLBeacon],
    in region: CLBeaconRegion) {
    
    guard let nearestBeacon = beacons.first else { return }
    let major = CLBeaconMajorValue(truncating: nearestBeacon.major)
    let minor = CLBeaconMinorValue(truncating: nearestBeacon.major)
    
    switch nearestBeacon.proximity {
    case .near, .immediate:
        displayInformation(for: major, and: minor)
        
    default:
        handleUnknownOrFarBeacon(for: major, and: minor)
    }
}
```

# How to adapt app’s behavior to location?
Date for matter provides localized presets and config options for user-visible representations of dates and times

```swift
// Formatting Dates
let dateFormatter = DateFormatter()
dateFormatter.dateStyle = .medium
dateFormatter.timeStyle = .short
dateFormatter.string(from: Date())
// "Jun 25, 2020 at 9:41 AM"
```

```swift
// Configuring the Format of Currency
let formatter = NumberFormatter()
formatter.currencyCode = "CAD"
formatter.numberStyle = .currency
formatter.string(from: amount)
// "CA$1.00"
```

Localize your app to appeal to users who speak a variety of languages
Add, export, and import localizations in Xcode

[[Localization-friendly Layout patterns]]
[[Creating Great Localized Experiences with Xcode 11 - 19]]
[[New Localization Workflows in Xcode 10 - 18]]


