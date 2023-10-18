# Built-in behaviors
Layout
* windows with light mode style
* iPad variant preferred
* landscape orientatin preferred
* will rotate into portrait if required
* scaling automatically provided

Input and interaction
* tap fingers
* look
* may touch content directly
* or bluetooth trackpad or game controller

local authentication auotmatically supports optic id.  No changes required.


# Functional differences
No notion of rotating the entire device.  You may want to specify which orientation your app prefers for new scenes.
```swift
UIPreferredDefaultInterfaceOrientation
```

default orientation for new scenes
no-op on iOS and iPadOS

```swift
UISupportedInterfaceOrientations
```
rotation supported if multiple orientations supported

```swift
UIRequiredDeviceCapabilities
```
determine if compatible.

manage availability fora sc

[[Explore App Store Connect for spatial computing]]

gestures.
* maximum of two simultaneous inputs
	* each hand is distinct touches
* system gestures of 2 touches or fewer work
* custom gestures may need updates

ARKit.
* ARView
* ARSession

existing views and sessions won't work on this device as they do on iPad/iPhone.
[[Re-imagine your ARKit app for spatial experiences]]

Location.
* approximated via wifi
* shared via iPhone
[[Meet Core location for spatial computing]]

look to dictate.
* avialable on search bars
* disabled by default
* No-op on iOS and iPadOS
* 
```swift
// SwiftUI
@State private var searchText = ""

var body: some View {
    NavigationStack {
        Text("Query: \(searchText)")
    }
    .searchable(text: $searchText)
    .searchDictationBehavior(.inline(activation: .onLook))
}


// UIKit
let searchController = UISearchController()
searchController.searchBar.isLookToDictateEnabled = true
```

best practices
* use availability checks
* check if frameworks are available
* verify hardware configurations are present and supported
	* ex not all cameras are available for apps to use.

xrOS device (designed for iPad)
added autoamtically for iOS SDK

# Choose your experience

Most apps don't need any changes!  Whether you want to stick with iOS SDK or rebuild against xrOS depends on your goals.

|                  | Designed for iPad      | designed for xros |
| ---------------- | ---------------------- | ----------------- |
| ---------------- | ---------------------- |  |
| SDK Availability | spritekit, storyboards | ARKit, REalityKit                 |
| scene types      | window                 |                window, volume, immersivespace |
| look and feel    | light mode             |        xros style           |
| ornaments        | unavailable            |          supported for windows         |



ornaments anchor to the side of the window to enhane its functionality.

[[Meet SwiftuI for spatial computing]]
[[Meet UIKit for spatial computing]]

* try your app!
* double-check compatibility areas
* audit info.plist keys
* opt into new platform experiences
* verify framework availability
* 
