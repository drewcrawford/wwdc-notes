Discover the App Store Server Library and learn how you can take advantage of resources and configurations for your apps. We'll show you how to set up the library, call the App Store Server API, verify App Store Server Notifications, and use app receipts. Explore insights and best practices for using App Store Server API endpoints, verifying App Store signed data, and migrating away from verifyReceipt.

New library to generate a JWT, verifyReceipt, etc.

app store, IAP, etc.

StoreKit 2
app store server api
app store server notifications v2


These tools provide transactions in a signed JWS format.  Provide developers robust info, client and server side.

App Store Server Library.
This library provides a set of functions to make it easier to adopt and integrate latest APIs available today and in the future.

Supported languages
* swift
* java
* node
* python

GitHub
* feedback and contributions are encouraged!

four key capabilities
app store server api.  By streamlining JWT creation, use a dozen different endpoings the server API offers
Verify JWS signed data, ensuring your transactions and server notifications are generated/signed by apple.
Extract receipt transaction ID, eliminating need to verifyReceipt endpoint.
Create promotional offer signatures.

[[Subscription Offers Best Practices - 19]]

# App store server API

Get transaction history endpoint.  By using a transaction ID, this API provides a customer's complete IAP transaction history.  A dozen endpoints.  All of which require a form of authentication. JWON web token.

Generating JWT is a critical step.  That's where the library comes in.

Starting in ASC... go to users and access module.
Keys.
IAP.
Issuer ID
Generate new private key.
Two pieces of information: key ID, and download the private key.  Downloading is only possible once.

Switching to apple PKI, see apple root certificates section.  Download all root certs.

Gradle demo.

Nothing more critical than storing iap private keys securely.
Generate a new key in ASC any time.

Utilize sandbox environment for development
Check regularly for updated apple root CAs.


# Signed data verification

What does signed data contain?
StoreKit signed data means it was generated and signed by the appstore in a JSON web signature format.  Data about iapp purchases, 

JWSTransaction
JWSRenewalInfo
AppTransaction -> originally purchased and currently installed
App Store server notifications v2.  Notification is signed data, and may contain various signed data payloads.

This JWS signed data is only in SK2, app store server API, App Store Server notifications v2 api.

Recommended to verify JWS signed data when
* delivering purchases on device
* receiving signed data
* App Store Server API responses

Cryptographic validation is challenging
Use a library like the App Store Server Library

Header is a json structure with fields defined by jws specification.  Our header has two fields: algorithm, ES256.  x5c, array of certificates to calculate expected public key that signed the JWS.

First cert is the leaf certificate.  It signed the JWS.  To verify it's from apple, construct chain of trust back to a known trusted source, e.g. apple authority.

Intermediate cert: apple worldwide relationships.  More specialized version of the root certificate authority.

Last cert is some apple authority.  Verify the certificate exactly matches a root we previously obtained from the infrastructure website.

Validate that these certs are from apple, etc.  To verify leaf certificate, check its purpose by looking for OID for mac app store receipt signing

for intermediate cert, check OID for devrel.

Check if cert has been revoked before engaging fverification process.  RFC 6960.

After verifying the cert chain, check JWS is signed by cert public key.  Extract leaf cert public key, JWs< and pass to JWS signature verification function.

Process is almost complete but there's one more thing.  

Also check the notification is targeted at your app and environment.  appAppleId, bundleId.  Check environment matches expected environment.  Just like other steps, our library checks these.

and that's how you do it.

Now let's see the library.

I don't require an app apple ID, so we pass null.  

onlineChecks let us know if we expect the thing to be current or if it's ancient history and dates may be wrong.

## Best practices

* `SignedDataVerifier`
* Confirm app identifiers within signed data
* As certs expire and can be revoked, don't hardcode certificates.  Validate that they are active.
# Moving to app store server API
App Store Server Library offers a utility to help with migration and ensure no app is left behind.

supports a wide array of capabilities:
Supports purchase validation, and contains endpoints for customer support, appeasement, etc.

New properties only added to JWS signed data.
Record original transaction ID, no more app receipts
NO longer need to save base64 encoded receipts
**Deprecation of the verifyReceipt endpoint.**

[[What's new in App Store Server APIs]]

Let's walk through flow diagram for app receipts.  While sk2 is a great tool to use, important to support clients on older device,s users that have not updated recently.

I'll show how these devices worked previously, how to continue supporting these clients following deprecation.

1.  Device sends receipt to your server
2. server passes receipt to verifyReceipt
3. receives decoded receipt.  Containing originalTransactionID
4. new request
5. server returns signed transactions.

okay now
1.  Directly extract transaction id from receipt
2. pass to API
3. store the revision from the endpoint.  Removes need to parse history each time.
4. Many of our endpoints including get transaction history now support any transaction id as a parameters.

List of transactions via API.  Could be decoded with signedDataVerifier from previous demo.

# Wrap up

See GitHub
docs, PRs, examples, etc.

* Download app store server library beta "soon"
* adopt it
* share feedback "and on github"

# Resources
* https://developer.apple.com/documentation/appstoreserverapi
* https://github.com/apple/app-store-server-library-java
* https://github.com/apple/app-store-server-library-node
* https://github.com/apple/app-store-server-library-python
* https://github.com/apple/app-store-server-library-swift
* https://www.apple.com/certificateauthority/
* https://developer.apple.com/app-store/subscriptions/#promotional-offers
* https://www.rfc-editor.org/rfc/rfc6960
* 