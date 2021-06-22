# Great features *and* privacy
* Data minimization => what is the value of this data?
* On-device processing => Why does the server need to see this data
* Transparency and control => What are we going to do with this data?
* Security protections => how is this data protected?


# Data minimization
iOS 14, "allow once"

In iOS 15, "Share my current location"
[[Meet the Location Button]]

Adopting the location button is a great way to build trust.

Location permission is session-based and will last until they leave the app.

Great replacement for "while in use"
Button customized to fit in with the rest of the app.  Background color, rounded corners.  Consistent string and iconography.  iOS can confirm that the button was tapped directly.

WatchOS, iOS, iPadOS, Catalyst

More precise control over when location is used.

Secure paste
* Confirms the paste button was tapped directly
* Transparency for pastes the user did not initiate

Notice only shown if the user did not initiate the paste.  Shift towards user-initiated paste.


# On-device processing
Main benefit is building amazing features with sensitive data.

* Siri

* No audio is stored by default
* Random identifiers
* On-device processing

Audio does not leave your iPhone

How designing for privacy enables great new features.  
Siri is incredibly fast.  By processing on-device, Siri can respond faster.

## Train modesl on-device in iOS 15
Apple Neural Engine powers moving more components of Siri on-device
Use CreateML to train models on-device in iOS

[[Build dynamic iOS apps with Create ML framework]]
# transparency and control

Building understandable controls helps build trust with people.

## Email
You may have thought twice before providing your email address to a service.

Hide My Email
Same easy address generation, new places.  Safari, mail, etc.
Addresses managed in Settings.

Make sure to respect their decision.

Remotely hosted images.  iOS privately load remote message content, hiding mail activity.

Using remote images, there are a few changes to be aware of.

## Indicators
Request access to read people's availability.  

* Put a purpose string in your info.plist
* Use state as a signal to others

[[Send communication and Time Sensitive notifications]]

## Recording indicator on Mac
* Indicator for mic access
* Behave the way people would expect
* Check on your SDKs

## App transparency
Privacy report
* New section in settings
* Shows app accesses to data
* Shows app contacts to domains
* Available in later seed

Found in privacy settings
off by default

Will not record private browsing mode in sfaari

Record app activity
* Same info as app privacy report
* Great way to review your data

List of stream dictionaries, for each time a sensor was accessed.  Search for all instances of bundle IDs.  

To see all domains, see final dictionaries.  Bundle IDs for keys, so one dictionary per executable.

Can see all domain contacts, when initiated, and if they're app or user initiated.

## Is it a website?

SafariViewController or ASWebAuthenticationSession are tagged by website.

Manually tag if
* contact is made on behalf of third party content
* Not app functionality

Set attribution to `.user` at standard layers
* `NSMutableURLRequest`
* `NWConnection`
* Sockets

Pass NSURLRequests to `WKWebView`

[[Explore WKWebView additions]]

> If you don't specify, connections will be marked as app traffic.

Added new methods to pass NSURLRequest.

For more info, see the talk.

## Record app activity and you
* App activity is becoming transparent
* Access expected data
* Access at expected times
* Build trust
* Understand SDKs

## Privacy labels

App privacy improvements
In late 2020, privacy nutrition labels were designed.

Best work can be invisible.  Your label is a place you can show off the hard work you do.

* Nutrition labels are not tied to OS version
* Align with application tracking transparency

* REquest tracking permission
* Makes it easier to trust new apps
* Requires a NSUserTrackingUsageDescription
* Educate before triggering the prompt
* Don't pre-prompt yes/no
* [[Apple Human Interface Guidelines]]

* Tracking is about linking
* Prompt covers all tracking, including IDFA
* Fingerprinting is never allowed
* Functionality can't be gated on tracking permission
* Align prompt with Privacy Nutrition Labels

See "App Store Guidelines: User Privacy and Data Use"

If tracking is denied, all app functionality must still be available.

## SDKs
You're responsible for your whole app
Reflect behavior in privacy label
Align with transparency prompt
Check your SDK documentation

## Ads
[[Meet privacy-preserving ad attribution]]

* Conversion value + source app
* signing key changines
* view-through attribution
* private click measurement
* multiple postbacks
* IP address protection
* postback to developer


# Security protections

## CloudKit encryption
New encrypted assets: NSString, NSNumber, NSDate, NSData, CLLocation, NSArray

We will provide encryption and key management.  

```swift
// Device 1: Encrypt data before calling CKModifyRecordsOperation.

myRecord.encryptedValues["encryptedStringField"] = "Sensitive value"


// Device 2: Decrypt data after calling CKFetchRecordsOperation.

let decryptedString = myRecord.encryptedValues["encryptedStringField"] as? String
```

# Pillars joining together

Profile is built by websites etc.

Tracking can transition to physical space.  Leads peopel to limit what they do on your website.

TLS and encrypted DNS help protect behavior patterns.

* [[Enable Encypted DNS]]
* [[Boost performance and security with Modern Networking]]

## iCloud private relay
* All connections are encrypted
* IP addreses no longer identify you
* IP address location is protected
* No single company can see what you do

Basically you have an ingress and egress proxy.

### Ingress proxy
* accepts connection
* protects IP from other servers
* encrypts all internet traffic

### egress proxy

Accepts connections from the internet
?

## relay

> RSA-blinded signatures

Using public key from token server, private key from proxy verifies access permission.

Private relay access token server provides a bundle of different tokens.  Access tokens, etc.

Connection is built using multiple layers ofencryption.  Proxy is removed layer by layer.  Only my device can decrypt every layer.

Egress proxy randomly selects IP.  Prevents cross-side tracking.

Ingress proxy can pick my location.  Websites can provide local content in safari, while precise location stays hidden.  Ingress server converts my location to a vague area.  IP used from the general area.

When "Use country and timezone" is provided, no hint is provided.  IP address picked from ingress proxy.  Very broad area.  

# Privacy pillars
* Data minimization
* On-device processing
* Transparency and control
* Security protections

# Wrap up
* Encrypt new assets in CloudKid
* Share Current Location button
* Privacy nutrition label
* App privacy report






