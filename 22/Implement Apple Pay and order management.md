Apple Pay provides an easy and secure way for people to make payments in your iOS, iPadOS, and watchOS apps as well as on the web. We'll take you through the entire Apple Pay implementation workflow – including how you can signal support for Apple Pay, request payment and handling updates, and add order details at the end of a payment flow to help people track their purchases.

# Get started

Merchant identifier.  To create one, head to identifier section of apple developer portal.  Reverse-dns, begin with `merchant`.

Payment processing certificate.  Apple pay encrypts each payload with this certificate.  Decrypt/process on your end.

Talk to your payment service provider for specifics.

apps:
navigate to signing and capabilities tab, and add merchant identifier

web:
register domains in identifiers section of portal.
create an apple pay merchant identity certificate.  Certificate authenticates sessions.


# implement apple pay
PKPaymentAuthorizationController.canMakePayments.

display the button.  Select the best call to action and button type.  Display button prominently, above the fold and as the first payment option.  Fully localized, can be adaptive to user's settings.

UIKit -> PKPaymentButton.
type -> kind of payment, eg donate, continue
style -> light, dark

web: load the button script.  Type,s tyle, localization.  CSS to further customize dimensions.

PKPaymentRequest.  
* merchantIdentifier
* merchantCapabilities
* countryCode
* currencyCode
* supportedNetworks (order of preference)
* paymentSummaryItems

PKPaymentAuthorizationViewControllerDelegate.  Always responsible for dismissing payment sheet.  Always try to return errors to the user as early as possible.

SwiftUI -> PaywithApplePayButton.  

Handling updates
PKPaymentAuthorizationResult
* return the status, whether the payment succeeded or failed.
* If there is a validation issue, set errors ordered by importance
* Order details can also be set in this result

w3C PaymentRequest spec for web.
# build an order
In iOS 16, we introduced order tracking.  Customers can now see order details and tracking from within wallet.

Identifiers section of developer portal, create an 'order type identifier'.  IDs your org as an entity that provides order information.  Use a similar reverse-DNS style.

Order Type ID certificate.  Use to build order packages, update orders, and send notifications.

Encrypted payload.  We already know your server is sent payment info for processing.  To support order tracking, also include some details about the order you created.  Enable the device to async request the order from your server.  Server returns order package to device.

* order type identifier
* Order identifier
* Web service URL
* Authentication token (shared secret) between user's device and your server

on server, verify auth token.  Then return order package to device.  Attempt to prepare your order as soon as possible so it's available on request.  if your server fails, we'll do multiple retries with an exponential backoff.

## What is an order package?

JSON dictionary.  Images.  Inline image.  Refer to HIG for guidance on creating images.  Localized resources, like strings files.  Keep an eye out for total order size.

Manifest -> another JSON dictionary.  Key for an entry is a relative file path, and value is its sha256 checksum.

Order type ID
certificate private key

apple WWDR intermediate certificate
PKCS #7 detached signature of manifest

# update an order

1.  Device registers for updates
2. server sends push notification
3. receive push notification
4. request order
5. returns updated order package

How to indicate you support orders?  in your json, you need `webServiceURL`.  Also add `authenticationToken` which we use for updates.

Manage registrations.  Server must be able to handle add or remove registrations
Find devices that registered for updates to an order.
Find orders that a device registered for.

Notify devices when there's an update.  

Notify devices customization.  changeNotifications can be set to customize how it's delivered.
`enabled` -> always send, default
`disabledIfAppInstalled` 

Push notification is empty, it doesn't know which orders have changed.  use the registration information to find relevant orders.  Keep track of update times and include Modification tag.  Value is opaque to device.  Device provides it next time.

## Best practices
* Offer express checkout
* Use information provided by apple pay
* Be mindful of the order package size
* Update orders promptly
* Review the apple pay section of the HIG
[[22/What's new in Wallet and Apple Pay|What's new in Wallet and Apple Pay]]
# Resources

* https://developer.apple.com/documentation/passkit/apple_pay
* 
* https://developer.apple.com/documentation/apple_pay_on_the_web
* https://applepaydemo.apple.com/
* https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentrequest
* https://developer.apple.com/documentation/walletorders/example_order_packages
* https://developer.apple.com/design/human-interface-guidelines/apple-pay/
* https://developer.apple.com/documentation/walletorders
