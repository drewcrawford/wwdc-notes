My pleasure to share with you an important security feature for managed devices in enterprise/education environments.

Your users need to access organization resources.  Websites, application servers, databases.

Attackers want to access those resources.  Perimeter security.  This model hasn't kept up with the way people interact with modern organizations.  Cloud providers put resources outside the perimeter.  Threats start inside the perimeter.  Spoof legitimate clients to penetrate the perimeter.  A modern security model does not start the network perimeter.

Trusted evaluation, zero-trust architecture.  Think of trust evaluation as a function.
Input => posture information
output => decision to grant or deny access.  Critical to get trust evaluation right.

a false negative gets in the way of user activities.  Or worse, a false positive allows an attacker to acces resources.  Accurate posture information is critical.

* user identity
* device identity
* location
* connectivity
* time
* device management
* etc

Low-security resources may require only a few items, vs all items sometimes.

User identity => who is making teh request.  Apple devices have several technologies.
* extensible SSO, kerberos
* enrollment SSO, app facilitate during enrollment

This session is about *device* identity.  Indicates which device is making the request.  
* UDID in MDM protocol
* `DeviceInformation` query
* Certificates issued to the device
* Implicitly trusts the device
* Security needs have evolved

# Managed Device Attestation
Device provides strong evidence about itself while making a request.  Improves posture to trust evaluations based on that are more accurate.

Legitimate devices reliably access resources, and attackers are thwarted.

# Attestation
Declaration of a fact.  If you trust the entity making the claim, you accept that the fact is true.

In software, an attestation is a fact that is *cryptographically signed*, usually x509.  if you trust the *signer*, you accept that the fact is true.

* device identity
* device properties

Only requires trusting the secure enclave, and apple attestation servers, whihc access manufacturing records, and operating system catalog.

DeviceInformation MDM command
ACME payload

Device identity attestation for broader organization.

## DeviceInformation
* `DeviceInformation` query
* obtain and return attestation on device, retunrned to server
* server evaluates attestation

Be careful.  Device information attestation declares to MDM server, "A device exists with these properties".  Doesn't prove the device **is** that device.  For that,

## ACME payload attestation
* strongest proof of the identity of the device
* configures client certificate, similar to SCEP
* ACME server receives attestation
* ACME server issues new certificate

* genuine apple ahrdware
* device identity
* Device properties
* Hardware-bound keys
* Correlation
# Benefits
A compromised device lies about its properties
apple attests to device properties

Compormised device provides outdated attestation
nonce ensures freshness

Device lies about which device it is
apple attests to device identities

Attacker extracts private key  from legitimate device
Apple attests that private key is hardware-bound

Attacker requests certificate for wrong device
Apple attests to identity of requesting device

How do you use them?
# Implementation
`DeviceInformation` => DevicePropertiesAttestation
Request attestation by adding to the `Queries` array
`DeviceAttestationNonce` key.
Nonce is optional, maximum 32 bytes

Response contains `DevicePropertiesAttestation`
array of data
attestation certificate chain

```xml
// DeviceInformation attestation request

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>RequestType</key>
	<string>DeviceInformation</string>
	<key>Queries</key>
	<array>
		<string>DevicePropertiesAttestation</string>
	</array>
	<key>DeviceAttestationNonce</key>
	<data>
	bWFnaWMgd29yZHM6IHNxdWVhbWlzaCBvc3NpZnJhZ2U=
	</data>
</dict>
</plist>
```

OIDs.

| OID                       | value         | device identifying |
| ------------------------- | ------------- | ------------------ |
| 1.2.840.113635.100.8.9.1  | Serial number | Yes                |
| 1.2.840.113635.100.8.9.2  | UDID          | Yes                |
| 1.2.840.113635.100.8.10.2 | sepOS version | No                 |
| 1.2.840.113635.100.8.11.1 | nonce         | No                   |

sepOS => OS that runs on the secure enclave.

## Validating the attestation
**in this order**
1.  Root cert is expected.  apple.com/certificateauthority/private/
2. Verify that the nonce matches
3. Parse and evaluate remaining OIDs

## Rate limiting
One new `DeviceInformation` attestation every 7 days
Specify `DeviceAttestationNonce` to request a fresh attestation
Omit `DeviceAttestationNonce` if freshness is not a concern
Cached attestation can be returned
A mismatched nonce may be expected.  Due to caching.

* don't request a new attestation as soon as possible (e.g. each 7 days)
* Request a fresh attestation when a property changes
* Occasionally request a fresh attestation

## Failed attestation
* devicePropertiesAttestation field is omitted
* Expected OID is missing or blank
* Network issue
* Attestation server issue
* Modified device hardware
* unercognized or modified software
* device is not a genuine apple device

Reacting
* no trustworthy way to know why it failed
* Don't always assume the worst when attestation fails
* Failure can affect the device's trust score
* Deny access or remediate depending on the trust score

## ACME payload
Start with an iPhone, iPad, or apple tv.
in most cases, this is managed by an MDM server.  There's an ACME server.  Implements the ACME protocol, RFC 8555.

Apple's attestation servers.

1.  MDM server installs profile on device, containing an ACME payload
2. payload specifies

`com.apple.security.acme`
* key properties
* certificate request
* ACME request (url, etc.)

3.  Device generates the requested type of key

* attestation requires a hradware-bound key
* ACME payload supports RSA or ECSECPrimeRandom.  But to get hardware key you need the latter.
* ECSECPrimeRandom, 256 or 384-bit key.

4.  Device makes initial contact with ACME server
5. Requests servers DirectoryURL
6. Account and order creation
7. Server offers `device-attest-01` validation type
8. Generate nonce in `token` field, sends to device

datatracker.ietf.org , device attestation extension to the ACME profile.

9.  Device requests an attestation from apple, nearly identical to the other attestation.,  
 
SAME oOIDs, device-identifying OIDs may be omtitted for user enrollment
nonce is hashed with SHA-256 first, the nonce comes from the ACME server, not from the MDM server
Private key is the one the device just generated.

Attestation certificate matches the private key, hwoever the certificate can only be used for attestation.

10.  Device requests another cert, this one *is* good for TLS.

Device provides CSR based on the payload
and attestation chain
and ClientIdentifier from ACME payload.

11.  Server validates rqeuest **in this order**
	1.  ClientIdentifier is valid and unused
	2. Cert muts chain up to Apple Enterprise Attestation Root CA
	3. Public key matches CSR
	4. SHA-256 hash of nonce matches
	5. Evaluate remaining OIDs
	6. Handle failed attestations

12.  ACME server issues client certificate and returnst o device

* Certificate is not obligated to match the payload
* Certificate stored in the keychain
* Payload installation is complete

Because client uses the same, hardware-bound key, we know these actions were performed by the same device.  And we have attestation certificate which describes that device.

Reference ACME payload's PayloadUUID
MDM IdentityCertificateuUID
WiFi
VPN 
kerberos
safari
many more

Up to 10 attested ACME payloads installed at the same time
Hardware-bound keys are **not** restored when restoring backups, **even** when restoring to the same device
Use ACME for MDM client identity

# Wrap up
* remediate multiple classes of threats
* Improve device posture
* Prove device identity


```xml
// DeviceInformation attestation response

<!-- ... -->
	<key>QueryResponses</key>
	<dict>
		<key>DevicePropertiesAttestation</key>
		<array>
			<data>
			MIIC0TCCAli <!-- ... --> pIbnVw= <!-- Leaf certificate -->
			</data>
			<data>
			MIICSTCCAc6 <!-- ... --> wjtGA== <!-- Intermediate certificate -->
			</data>
		</array>
	</dict>
<!-- ... -->
```

* https://developer.apple.com/forums/tags/wwdc2022-10143
* https://developer.apple.com/forums/create/question?&tag1=93&tag2=466030
* https://apple.com/certificateauthority/private/
* https://datatracker.ietf.org/doc/draft-bweeks-acme-device-attest/
* https://developer.apple.com/documentation/devicemanagement
