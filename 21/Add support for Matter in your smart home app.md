#matter
#homekit 

# Matter
* A new standard for smart home accessories
* Interoperability among devices
* Leverages HomeKit technology

# Benefits
* Simplifies develompent effort for acessories and apps
* Allows more choices for customers
* Provides builders a standard set of accessories to pre-install
* Coming in iOS 15 as a developer preview

# Integrate with Matter via HomeKit

## Architecture
Today, you implement HAP protocol.  We can expand HomeKit to support Matter in parallel.

CHIP (former name for Matter).  We expect to have updated naming in a later release.

All existing apple home features to begin working with matter seamlessly and immediately.  

HK responsibility isn't just first-party features.  In fact, since day 1, we designed HK to build creative smartphones apps.

Starting in iOS 15, `home.addAndSetupAccessories()` will start recognizing Matter QR codes.

## Developer preview
* A consistent and familiar flow for accessory setup
* Accessory control and configurations
* Remote access and timely notifications
* Scenes and automations

## Future support
* More Matter accessory categories
* Custom features



# Support for other smart home hubs
New yet familiar APIs to establish a direct connection between a matter accessory and your home hubs.

Whiel this is system UI, the homes and rooms the user selects from are vended from your app.  You have to provide the information via new app extension type.

## Invocation
Create topology object and pass to iOS

```swift
let homes = proprietaryHomeStorage.homes.map { home in
    HMCHIPServiceHome(uuid: home.uuid, name: home.name)
}

let topology = HMCHIPServiceTopology(homes: homes)
let setupManager = HMAccessorySetupManager()

do {
    try await setupManager.addAndSetUpAccessories(for: topology)
    print("Successfully added accessory to my app”)
} catch {
    print("Error occurred in accessory setup \(error)")
}
```

User is now presented with the option of adding the accessory to your app.

If your topology consisted of only a single home, this is skipped.  Once user selects a home, iOS selects an extension

```swift
class RequestHandler: HMCHIPServiceRequestHandler, CHIPDevicePairingDelegate {

   // . . .

   override func pairAccessory(in: HMCHIPServiceHome, onboardingPayload: String) async throws -> Void {
        // iOS is instructing the extension to pair the accessory via CHIP.framework
  }     // . . .
}
```

Fetch room request/response

```swift
class RequestHandler: HMCHIPServiceRequestHandler, CHIPDevicePairingDelegate {

   // . . .

   override func rooms(in: HMCHIPServiceHome) async throws -> [HMCHIPServiceRoom] {
        // iOS is querying for a room list that corresponds to the given home
    }     // . . .
}
```

Once name and room are selected, provide that to your extension

```swift
class RequestHandler: HMCHIPServiceRequestHandler, CHIPDevicePairingDelegate {

   // . . .

   override func configureAccessory(named accessoryName: String, room accessoryRoom: HMCHIPServiceRoom) async throws -> Void {
        // iOS is instructing the extension to apply configuration via CHIP.framework.
    }     // . . .
}
```

User is offered the opportunity to use apple stuff to control the accessory

Here's all the functions

```swift
class RequestHandler: HMCHIPServiceRequestHandler, CHIPDevicePairingDelegate {

    override func rooms(in: HMCHIPServiceHome) async throws -> [HMCHIPServiceRoom] {
        // iOS is querying for rooms that match the given home.  These rooms will be shown in system UI and the selection will be vended back to your extension's `configureAccessory` function
   }

    override func pairAccessory(in: HMCHIPServiceHome, onboardingPayload: String) async throws -> Void {
        // iOS is instructing the extension to pair the accessory via CHIP.framework
   }
    
    override func configureAccessory(named accessoryName: String, room accessoryRoom: HMCHIPServiceRoom) async throws -> Void {
       // iOS is instructing the extension to apply configuration via CHIP.framework.
    }
}
```

UI presentation, transitions, error handling,e tc., are all handled for you.  This reduces the amount of code you need to write.

# A deep dive into Matter protocol
If you're an accessory developer, you will need to get familiar with the protocol itself.

## Data model
* Endpoints.  Logically separate features of an accessories.  Information (name/vendor/model), function
* Each endpoint can have multiple clusters.  Light endpoint might have brightness, power control, etc.  Similar to HomeKit service.
* Attribute, represents some state of the endpoint.  Similar to HomeKit characteristics
* Each attribute may support a set of actions from read/write/report.  

## CHIP.framework
* Developed in open source
* Renamed to Matter
* Distributed in iOS

## Setup and control
```swift
let controller = CHIPDeviceController.shared()
do {
    let device = try controller.getPairedDevice(accessoryDeviceID)
    let onOffCluster = CHIPOnOff(device: device,
                               endpoint: lightEndpoint,
                                  queue: DispatchQueue.main)
    onOffCluster?.toggle({ (error, values) in
        // Error handling code here
    })
    onOffCluster?.readAttributeOnOff(responseHandler: { error, response in
        if let state = response?[VALUE_KEY] as? NSInteger {
           updateLightState(state: state)
        }
    })
} catch {
    print("Error occurred in accessory control \(error)")
}
```

Read the state of the attribute, accessory sends us the response and we update our light state.

## Multiple home hubs
* List connected home hubs
* Manage connected home hubs
* Allow a new home hub to connect

## Developer preview
* A developer profile to enable Matter support in iOS/iPadOS/tvOS 15
* A home hub required to control Matter accessories via HomeKit
* Sample VendorID+ProductID for supported accessory categories
* More information about Apple APIs on developer.apple.com
* More information about Matter open source APIs on GitHub

# Wrap up
* Seamless support for accessory developers, HomeKit apps, and HomeKit users
* A new setup API (in an upcoming seed) to connect Matter accessories with other smart home hubs
* Available in iOOS/iPadOS/tvOS 15 developer preview

