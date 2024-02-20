Discover the latest updates to Wallet and Apple Pay. Learn how to take advantage of preauthorized payments, funds transfer, and Apple Pay Later merchandising to create great Apple Pay experiences in your app or for the web. Explore improved support for Mail, Messages, Safari, and third-party apps in Wallet Order Tracking, and find out how you can add more information to an order's transaction or receipt details. And we'll introduce you to Tap to Present ID on iPhone (or ID Verifier), a new way to accept IDs in Wallet using iPhone — no additional hardware needed.

### Performing a display request to verify age - 28:51
```swift
import ProximityReader

// Check the current device supports mobile document reading.
guard MobileDocumentReader.isSupported else { return }
    
let reader = MobileDocumentReader()
    
let readerSession: MobileDocumentReaderSession = try await reader.prepare()

let request = MobileDriversLicenseDisplayRequest(elements: [.ageAtLeast(21)])

try await readerSession.requestDocument(request)
```

### Displaying brand information during a document request - 30:55
```swift
let reader = MobileDocumentReader()

let identifier = try await reader.configuration.readerInstanceIdentifier
let readerToken = try await WebService().fetchToken(for: identifier)

let readerSession = try await reader.prepare(using: .init(readerToken))

let request = MobileDriversLicenseDisplayRequest(elements: [.ageAtLeast(21)])

try await readerSession.requestDocument(request)
```

### Performing a data request - 31:50
```swift
let session: MobileDocumentReaderSession = /* ... */

var request = MobileDriversLicenseDataRequest()
request.retainedElements = [.givenName, .familyName, .dateOfBirth, .portrait]
request.nonRetainedElements = [.address, .documentExpirationDate, .drivingPrivileges]

let response = try await session.requestDocument(request)

// Process document elements from document response.
self.processResponse(response.documentElements)
```
# Resources
* https://developer.apple.com/documentation/ProximityReader/adopting-the-verifier-api-in-your-iphone-app
* https://developer.apple.com/documentation/ProximityReader/generating-reader-tokens-for-the-verifier-api
* 