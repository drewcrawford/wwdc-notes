Allows users to connect to their providers and securely download records.

FHIR.

[[Accessing Health Crecords with HealthKit - 18]]

Allow download, storage, sharing of verifiable records based on "smart health card" specification.

# Verifiable Health Records

* Bundle of patient and clinical resources
* Packaged as a JWS
* Data minimization

JWS has 2 components, header and payload.
Header => algorithm, public key id, zip (algorithm)

issuer
nbf => "not before date"?
expiration date

Visit "smart healthcard specification"

How to import into health app?  Import into existing records.

Connect to health records enabled provider.  Only available where health records feature is enabled.  US, UK, canada.

`*.smart-health-cards` extension, or scan a QR code.

Users can view details and choose whether to download these records.  Available internationally.

How to request access with HK?

`HKVerifiableClinicalRecordQuery`
`HKVerfiableClinicalRecord`.

Unlike other HKQuery subclasses, you must request a HK access entitlement for verifiable records.

Results handler will return an authorizationDenied error without the entitlement.

# Authorizaiton
* Per-sample authorization

`healthStore.requestAuthorization(...)
let query = HKSampleQuery...
`

With verifiable records, this isn't needed.  You are prompted when the query is executed.

After sharing, records are returned to query's results handler.

Auth is one-time only.  Sharing with a 3rd-party app does not setup long-term auth.

Each execution of a query presents the auth sheet.

# HKVerifiableClinicalRecordQuery
Provide record types
Only records with all types will be shown in the authorization sheet
Predicate by relevant date
Receive HKVerifiableClinicalRecords
Verify jwsRepresentation

# New ways to import
Health records
Download a `.smart-health-card` file
QR code
```swift
import HealthKit
let leathStore = HKHealthStore()
let recordTypes = ["https://smarthealth.cards#immunization"]

let endDate = Date()
let startDate = ...
late dateInterval = ..
let predicate = predicateforVerfiiableClinicalRecords(...)
let query = HKVerifiableClinicalRecordQuery(...)
healthStore.execute(query)
```

Digital sigantures are a mathematical way to verify authenticity of data.

# Verify the signature
* Codable to parse
* Compression framework to decompress
* Combine to download keys
* CryptoKit to verify the signature

```struct JWS: Codable
headerString
header
payloadString
...
```

```swift
struct JSWHeader: codable {
}
```

1.  Decode json, base64
2.  Check if th eheader indicates that the payload is compressed correctly
3.  See sample project fo rhow to decompress payload

```swift
struct SMARTHealthCardPayload: Codable {
	let iss: String
	...
	let vc: VC
}
```

For VC, see
[[Handling FHIR without getting burned]]

```swift
extension JSW {
	public func verifySignature() -> AnyPublisher<Bool, Error> {
		let issuerIdentifier = decodedPayload.iss
		guard URLIsTrusted(...)
		let endpointString = issuerIdentifier + "./well-known/jwks.json"
		let url = URL(string: keyEndPointString)!
	}
}
```

download their keys, finish verifying the signature.

URLSessiondatataskpublisher, map to data, decode, check if signature is valid.

Use cryptokit to verify signature.  See sample project.

# Privacy
* Data minimization
* Data encryption
* Access entitlement required
* Granular control

# Wrap up
* Learn more about SMART Health Cards
* Import test records
* Download tehs ample app
* Request entitlement

* https://developer.apple.com/documentation/healthkit/hkverifiableclinicalrecordquery
* https://developer.apple.com/documentation/healthkit/hkverifiableclinicalrecord
* https://developer.apple.com/documentation/healthkit/samples/accessing_data_from_a_smart_health_card
* https://developer.apple.com/documentation/healthkit/samples/accessing_a_user_s_clinical_records
* https://developer.apple.com/documentation/healthkit/samples/accessing_health_records

