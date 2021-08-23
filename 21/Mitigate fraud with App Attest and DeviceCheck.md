#devicecheck
# Trust and safety
DeviceCheck framework
* DeviceCheck (iOS, macOS, tvOS)
* App Attest (iOS, tvOS)

[[Safeguard your accounts, promotions, and content]]

# One-time promotion?
You want to limit promotional items to once per device.  
# DeviceCheck
* Two bits as device state stored by Apple (and a timestamp)
* Per device, per developer.  Applicable across all your apps.
* Persistent across device resets.  Consider using timestamp to reset this on a period of time at your choosing.

[[Privacy and your apps - 17]]

# App Attest
* Validate instances of your app running on a device

Modified apps, users cheat, etc.  Spoof location.  Fake app traffic.

* Genuine apple device
* Authentic application identity
* Trustable payload

We include a hash of the app identity in the attestation.  By comparing, you can determine if you're using a modified version.

Request payload.  Can instruct app attest to attach assertion to payload.  Your app transmits payload and assertion to servers.  By verifying assertion, trust that the payload was not tampered with in transit.

## Privacy
* Attestations are anonymous
* Keys are unique per installation.  App attest key will not survive at reinstallation.  Not synced across devices.

## Steps to integrate
1.  Create an App Attest key
2.  Attest and verify key
3.  Generate and verify assertions

```swift
let appAttestService = DCAppAttestService.shared

if appAttestService.isSupported {
    appAttestService.generateKey { keyId, error in
        guard error == nil else { /* Handle the error. */ }
        // Cache keyId for subsequent operations.
    }
} else {
   // Handle fallback as untrusted device
}
```

Is supported on devices with secure enclave.  But there are cases, such as app extensions, where `isSupported` will return `false`.  Must handle these gracefully.

* Use as signal for risk assessment
* Monitor spikes in unsupported devices

### Generate key attestation
```swift
appAttestService.attestKey(keyId, clientDataHash: clientDataHash) { attestationObject, error in
    guard error == nil else { /* Handle error. */ }

    // Send the attestation object to your server for verification.
}
```

Evidently apple is signing this stuff on their side and returning them to the device.

### Verify key attestation
1.  Certificates.  Leaf, intermediate.  Nonce.  Reconstruct nonce and verify it matches.
2.  Authenticator data.  Contains app identity hash.  Use this to verify it is your app.
3.  Risk metric receipt.  

Store the key associated with client data when used to verify subsequent requests
* Avoid penalizing users on failure (might be network-related?)
* Use failures as signals

Network only happens once per app instance, but if you have a large install base, you will send many requests.

### Plan for ramp up
Gradually enable across your install base.  If you have 1M users you can probably ramp up over a day or so.  For 1B, ramp up over a month or more.

### Verify assertions
```swift
appAttestService.generateAssertion(keyId, clientDataHash: clientDataHash) { assertionObject, error in
    guard error == nil else { /* Handle error. */ }
    
    // Send assertion object with your data to your server for verification
}
```

* Signature: Reconstruct nonce and use public key to verify signature.
* Authenticator data: Contains app identity hash.  Validate the hash to ensure it's from your app.  Also contains assertion counter.  To protect against replay attacks, store this on your server and expect it to increase with each request.

## When to use assertions
* State synchronization
* Establish sessions

not for
* Real time operations
* Low latency operations

## Risk metrics service

Attacker may use a single valid device to generate various keys, and then hand them to bots / allow bots to use.  For this, we have "App Attest risk metric service".

Periodically redeem the risk metrics with receipts.

* Payload
* Certificate chain
* Signature

## App clips
Added app attest to app clips in iOS 15.  App clip share the same app identity in app attest ?

App clips are manually removed or expired, their keys will be invalidated just like when your full app is uninstalled.

## Keys to success
* Importance of server-side validation
* One-time server challenges
* Failures as signals

# It's your turn
* https://developer.apple.com/documentation/devicecheck/establishing_your_app_s_integrity
* https://developer.apple.com/documentation/devicecheck/assessing_fraud_risk
* https://developer.apple.com/documentation/devicecheck/validating_apps_that_connect_to_your_server
* https://developer.apple.com/documentation/devicecheck/accessing_and_modifying_per-device_data

