Push notification service is only available when the device has a network connection.  Feedback suggests this must continue when no network connection.

# Recap push notifications
1.  Provider server to APNs
2.  APNs to device

Your app might use PushKit or push notification framework to receive.

Push notification offers the desired user experience and most importantly it is very energy-efficient.  Developers should prefer PushKit or the notifications framework.

However, if your app is used in a network environment where APNs cannot be reached, the app cannot receive notifications.  e.g., cruise ships, airlines, etc.  

Very important to keep PNs functional even in constrained network envrionments.  Local connectivity is designed to solve this problem.
# New API
App extension can directly communicate with your server.  Need to define your own protocol.

Your app must specify the wifi networks where you wish to enable connectivity.
App extension maintains connection and receives the notifications.  Runs in the background as long as the device is associated with the specified wifi networks.

## Guidelines
* Prefer apns for general use cases
* Use local push connectivity for specific use cases.  Mainly defined by limitations in network environments.
* enabled by a new entitlement

| Push notifications         | Local Push Connectivity          |
|----------------------------|----------------------------------|
| Requires APNs connectivity | Requires local user connectivity |
| Any network                | Specific Wi-Fi Networks          |
| Apple APNs protocol        | Custom protocol                  |

## user facing notifications
1.  UserNotifications framework in the app extension.

## VoIP notification
`NEAppPushProvider` in app extension.  System wakes containing app with CallKit.

## demo

# Local push connectivity API
* `NEAppPushManager`
* `NEAppPushProvider`
* `NEAppPush` entitlement

## `NEAppPushManager`
* specify wi-fi networks
* Handle incoming VoIP call

## `NEAppPushProvider`
* Life-cycle management of app extension
* Wake containing app on receiving VoIP notification




```swift
//create configuration
import NetworkExtension

let manager = NEAppPushManager()
manager.matchSSIDs = [ "Cruise Ship Wi-Fi", "Cruise Ship Staff Wi-Fi" ]
manager.providerBundleIdentifier = "com.myexample.SimplePush.Provider"
manager.providerConfiguration = [ "host": "cruiseship.example.com" ]
manager.isEnabled = true

manager.saveToPreferences { (error) in
    if let error = error {
        // Handle error
        return
    }
    // Report success
}
```

```swift
// Manage App Extension life cycle and report VoIP call 

class SimplePushProvider: NEAppPushProvider {

    override func start(completionHandler: @escaping (Error?) -> Void) {
        // Connect to your provider server       
        completionHandler(nil)
    }

    override func stop(with reason: NEProviderStopReason,
                                   completionHandler: @escaping () -> Void) {
        // Disconnect your provider server
        completionHandler()
    }
    
    func handleIncomingVoIPCall(callInfo: [AnyHashable : Any]) {
        reportIncomingCall(userInfo: callInfo)
    }
}
```


```
//handling incoming VoIP call in the containing app
class AppDelegate: UIResponder, UIApplicationDelegate, NEAppPushDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions:       
                     [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        NEAppPushManager.loadAllFromPreferences { (managers, error) in
            // Handle non-nil error
            for manager in managers {
                manager.delegate = self
            }
        }
        return true
    }
    
    func appPushManager(_ manager: NEAppPushManager,
                didReceiveIncomingCallWithUserInfo userInfo: [AnyHashable: Any] = [:]) {
        // Report incoming call to CallKit and let it display call UI
    }
}
```

Note that you must have the "Voice over IP" background mode enabled.

# next steps
* prefer push notifications
* Use Local Push Connectivity in certain conditions
* `NEAppPushProvider` requires entitlement

https://developer.apple.com/contact/request/network-extension-app-push-provider
https://developer.apple.com/documentation/callkit
https://developer.apple.com/documentation/networkextension/local_push_connectivity/receiving_voice_and_text_communications_on_a_local_network
