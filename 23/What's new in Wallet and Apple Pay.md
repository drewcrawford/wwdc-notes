Discover the latest updates to Wallet and Apple Pay. Learn how to take advantage of preauthorized payments, funds transfer, and Apple Pay Later merchandising to create great Apple Pay experiences in your app or for the web. Explore improved support for Mail, Messages, Safari, and third-party apps in Wallet Order Tracking, and find out how you can add more information to an order's transaction or receipt details. And we'll introduce you to Tap to Present ID on iPhone (or ID Verifier), a new way to accept IDs in Wallet using iPhone — no additional hardware needed.

# Payments
## Apple Pay Later
while no integration is required, we have a new UI for better experience
Indicates acceptance of APL
choose an appropriate style based on context
provides further information when selected
available for use in apps and on the web

four display styles
* standard.
* badge.  Support but no details
* checkout - placed alongside other payment options
* price -> use alongside total purchase price

learn more
calculator

Can use PKPayLaterUtilities.validate to provide amount and locale.  Once you determine user is eligible, instantiate a PKPayLaterView with same details provided to the eligibility check.

on the web, uses the same javascript SDK.
set the `crossorigin` attribute to support CORS requests.
Declare the `async` attribute when including the script
Requires a JWT, available from apple developer portal

Use `apple-pay-merchandising` element.  

Best practices:
For apps, an entitlement is required to use this view.
websites, register your domains and obtain the jwt

import JS file within the `<head>` tag to run asap.
Ensure the view meets sizing requirements.
set content security policy appropriately


## Preauthorized payments

in iOS 16 we introduced this.
* view and manage preauthorized payments in Wallet
* supports - recurring payments, automatic reload
* now support deferred
* available for use in apps and on the web

fixed or variable amount charged in the future
optionally provide a free cancellation cutoff date
* booking a hotel reservation
* pre-ordering an item

tied to a user's apple ID rather than individual device.
More reliably charge customers at a later date
automatically issued if the card's payment network supports them.

[[22/What's new in Wallet and Apple Pay|What's new in Wallet and Apple Pay]]


first we create PKDeferredPaymentSummaryItem
deferredDate
PKDeferredPaymentRequest
summary item we just created

to display billing agreement, can set on deferred payment request.

free cancellationdate
defines a time period where cancellation is free of charge
explicitly provide the timezone that applies to the cancellation date!

Customer's timezone might not match your cancellation policy's timezone.

Consider the date, time, and timezone for free cancellation
Include deferred payments in your summary items
keep billing agreement text short
only a summary of key facts, not replace normal agreements
specify a token notification URL

## Transfer funds with Apple Pay

Ability to transfer funds to a payment card
Uses the same apple pay infrastructure as payments
Examples include withdrawing funds from
* a bank account
* a stored value account

A dedicated request type that focuses on funds transfers
defines an amount to be transferred to a user's payment card
optionally you can ask for recipient details
similar requirements as PKPaymentRequest

[[Implement apple pay and order management]]

`supportsDisbursements`.  Provide networks and card capabilities to check for support.

summary item -> label - name of your business
PKDisbursementSummaryItem -> label -> amount received

PKDisbursementRequest.  

for transfers you receive the same type of payment for processing.  

convenience methods.  Use disbursementContactInvalidError for issues relating to contact information provided.  If you determine that the user's payment card is uanble to accept transfers, use unsupportedError.

* ability to represent instant funds transfers
* an associated instant transfer fee can be set
* provide a capability to require support
* limited to cards supporting instant transfers

`instantFundsOut` to supported capabilities.  Can check with `supportsDisbursements`.  Adjust transfer method options and UI accordingly.

transfer funds, only iOS/iPadOS.  Not macos/web
return an appropriate disbursement error if required
first summary item must be the amount withdrawn from source
label matches name of your business
last item must be amount to received (ex net of fees)

# Order tracking

## System integration

Support for maps.  If users  is tracking an order with pickup location, we suggest it.

New `shippingType` property.
Ability to define whether an order is being
* shipped
* delivered
allows for more suitable messaging in Orders.

Enhancements to better support associated applications
`associatedApplicationIdentifiers` to improve managemen tor order notifications
`customProductPageIdentifier` to reference a custom app store product page.

An array of payment transactions per order.
Associate a receipt with a transaction.  PDF or JPEG/PNG.
Ability to represent purchases and refunds.

Attach order packages to emails, such as order conformation email.  User can add order to wallet.  
"Track with Apple Wallet" button.

FinanceKit and FinanceKitUI
New Swift (exclusive!) frameworks to handle data within Wallet.

Access and management done via `FinanceStore`.
Check if order already exists
Add/update an order

If you want to add/update an order, can do it two ways.

FinanceKit -> serialized signed order package.
`saveOrder` method.  Screen will be presented for user confirmation.
* added
* cancelled
* newer order already exists

for swiftUI we use FinanceKitUI.  

can also support web.  `<apple-wallet-button` tag.  Configure with attributes.  `type="track-order"`.  Set onClock to point to the location of the signed order package.



# Identity
we introduced Ids back in iOS 15.4.  Users in supported US states to add their drivers license or ID to wallet.  Last year we introduced Verify with Wallet.  Businesses can streamline by requesting info stored in apple wallet.

This year, we're introducing 'Tap to present ID on iPhone'.  Wtih this API, your apps can seamlessly/securely verify ids in wallet or other mobile driver's licenses using just iPhone.  Builds on top of tap to pay which we added to proximity reader framework in iOS 15.4

display request
* verify age or name
* result displayed by iOS
* NOT returned to your app
data request
* wider set of elements
* result returned to app
* additional entitlement required


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

By default, brand name/logo isn't displayed on either side.  However if you want to display during request, you can.
### Displaying brand information during a document request - 30:55
```swift
let reader = MobileDocumentReader()

let identifier = try await reader.configuration.readerInstanceIdentifier
let readerToken = try await WebService().fetchToken(for: identifier)

let readerSession = try await reader.prepare(using: .init(readerToken))

let request = MobileDriversLicenseDisplayRequest(elements: [.ageAtLeast(21)])

try await readerSession.requestDocument(request)
```

reader token is a JWT, signed with a keypair you've configured through apple business register.  Your server creates the reader token with brandID, key ID, and reader instance identiifer.  The first two are the same across all instances of your app.  App provides the last one to your server.


Data request.  


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

# wrap up
* enroll in apple business register
* check out forums
* feedback assistant, etc.
* 
# Resources
* https://developer.apple.com/documentation/ProximityReader/adopting-the-verifier-api-in-your-iphone-app
* https://developer.apple.com/documentation/ProximityReader/generating-reader-tokens-for-the-verifier-api
* 