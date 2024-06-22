Location authorization is turning 2.0. Learn about new recommendations and techniques to get the authorization you need, and a new system of diagnostics that can let you know when an authorization goal can't be met.


###  CLLocationUpdate and CLMonitor - 0:31
```swift
// Iterating liveUpdates to reflect current location
Task {
    let updates = CLLocationUpdate.liveUpdates()
    for try await update in updates {
        if let loc = update.location {
            updateLocationUI(location: loc)
        }
    }
}
    
// Iterating monitor events to report condition state changes
Task {
    let monitor = await CLMonitor(monitorName)
    await monitor.add(CLMonitor.CircularGeographicCondition(center: applePark, radius: 50), identifier: "ApplePark")
    for try await event in await monitor.events {
        updateConditionsUI(for: event.identifier, state: event.state)
    }
}
```

Before you can use those apis, you need permission.

###  Handle updates with CLLocationManagerDelegate - 0:52
```swift
// Adapting location authorization to Swift with a MainActor singleton
@MainActor class LocationReflector: NSObject, CLLocationManagerDelegate, ObservableObject {
    static let shared = LocationReflector()
    private let manager = CLLocationManager()
    
    override init() {
        super.init()
        manager.delegate = self
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager){
        if (manager.authorizationStatus == .notDetermined) {
            manager.requestWhenInUseAuthorization()
        }
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations:[CLLocation]) {
        // Process locations[0]
    }
    // ...
}
```


###  CLServiceSession simplifies - 1:07
```swift
// CLServiceSession in action
Task {
    let session = CLServiceSession(authorization: .whenInUse)
    for try await update in CLLocationUpdate.liveUpdates {
        // Process update.location or update.authorizationDenied
    }
}
```

CLServiceSession and a new system of diagnostic properties.

# Authorization goals

1.  NotDetermined
2. When in use or always, approximate or precise.

Until now, you asked for auth using CLLocationManager.  
`requestTemporaryFullAccuracyWithPurpose` key.  But temp auth lapses.  What if they really only granted you whenInUse with allow once in the first place?

Now you can express authorization goals directly.  
###  Implicit service sessions - 7:15
```swift
// CLServiceSession in action
Task {
    let session = CLServiceSession(authorization: .whenInUse)
    for try await update in CLLocationUpdate.liveUpdates {
        // Process update.location or update.authorizationDenied
    }
}
```

Best practices: 
Specify goals not steps
Take proactively

Why would someone want to use approximate location?  What are the special features that request an exception?

Layer, don't modify


In many cases, if you're iterating live updates, you have a goal of receiving them.  So the act of iterating is implicitly holding a service session.  So you may be able to update just by deleting code.

Implicit service sessions
* enabled by default
* set a `whenInUse` goal

###  Implicit service sessions - 7:34
```swift
Task {
    for try await update in CLLocationUpdate.liveUpdates {
        // Process update.location or update.authorizationDenied
    }
}
```

So if you don't want API use to implicitly motivate authorization, can disable this in info plist.

So why not rely on implicit sessions?
* temporary full accuracy
* Always authorization -> soon only acceptable when you request the session
* control authorization request timing

# Session lifecycle

1.  Geotagging a photo
2. browsing mapkit view for awhile
3. overlapping features

When a feature continues conceptually, so will implicit sessions.  

Why hold sessions in the bg:
1.  Maintain your authorization goal
2. Be ready for return
3. Empower always authorization
4. Just more natural

Works even if suspended, works even when terminated.  
We don't take measures to keep apps running continuously, we don't send authorization-prohibited updates.  But we do keep track of each outstanding API object, like CLServiceSession, liveUpdates, events, CLBackgroundActivitySession.  Your resume or relaunch when more info is ready, etc.

So we're ready to come back when relaunching.

# Diagnostic properties

Tells you why you can't meet your goals.



###  Diagnostics – Following the progress of location authorization - 13:37
```swift
// Following the progress of location authorization with CLServiceSession
let mySession = CLServiceSession(authorization:.whenInUse)
for try await diagnostic in mySession.diagnostics {
    if (diagnostic.authorizationDenied) {
        // Ok, let’s let them pick a location instead?
    }
}
```

`insufficientlyInUse` -> you can't yet ask for authorization

`authorizationDeniedGlobally` -> if location services are disabled system-wide.  We treat these as two different properties.  So that if you only want to know if you're explicitly denied access, then it only needs to check the general one.  But if you want to provide different advice to globally-denied people, can check the more specific property as well.

###  Diagnostics – Following the progress of location authorization - 15:00
```swift
// Following the progress of location authorization with CLServiceSession
let mySession = CLServiceSession(authorization:.whenInUse)
for try await diagnostic in mySession.diagnostics {
    if (!diagnostic.authorizationRequestInProgress) {
        // They’ve decided (maybe already). We can move on!
        break
    }
}
```



###  Diagnostics – Following the progress of location authorization - 15:25
```swift
// Following the progress of location authorization with CLServiceSession
let mySession = CLServiceSession(authorization:.whenInUse)
for try await diagnostic in mySession.diagnostics {
    if (!diagnostic.authorizationRequestInProgress) {
        reactToChanges(authorized:!diagnostic.authorizationDenied)
    }
}
```

CLLocationUpdate and CLMonitorEvent also have diagnostic properties.

`accuracyLimited` -> only every 15-20 minutes
`locationUnavailable` -> unable to determine location.

CLMonitor.Event can also return some of these.

Now, if for some reason we can't give you the info you're expecting, we at least give a diagnostic explaining why not.

###  Diagnostics – Following the progress of location authorization - 15:46
```swift
// Following the progress of location authorization with CLServiceSession
Task {
    let mySession = CLServiceSession(authorization:.whenInUse)
    for try await diagnostic in mySession.diagnostics {
        if (!diagnostic.authorizationRequestInProgress) {
            reactToChanges(authorized:!diagnostic.authorizationDenied)
        }
    }
}
```

# Wrap up
* describe your authorization goals with CLServiceSession if needed
* Diagnostic properties will tell you if anything goes wrong
[[Discover streamlined location updates]]
[[Meet Core Location Monitor]]

# Resources
* https://developer.apple.com/documentation/CoreLocation/adopting-live-updates-in-core-location
* https://developer.apple.com/documentation/CoreLocation/configuring-your-app-to-use-location-services
