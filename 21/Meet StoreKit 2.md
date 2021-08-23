#storekit 

Focus on client-side features.

This year, we decided to go back to the beginning.  SK2.

# Meet SK2
* New set of Swift APIs for in-app purchase
* Updates to receipts and transactions
* More subscription APIs
* Same StoreKit framework

* Products
* purchases
* transactio ninfo
* transaction history
* subscription status

# Products & Purchases
* Product type
* subscription info
	* `isEligibleForIntroOffer`

Product=>`BackingValue`.  Always be able to access new properties even on SDKs that have older versions.  Use the latest features to provide new functionality to more customers.

thansk to new swift concurrency async/await, 1 LOC.

## Purchase options

Item that describes a single property of a purchase.  Compose into sets, pass to method.

* Quantity
* Promotional offer
* App account token

### App account token

Which user accounts began and completed a transaction
* You create it
* Link to in-app account
* UUID format
* Stored in trasanction forever, even across devices

# Transaction info
* One (individually-signed) object per transaction
* JSON - JSON web signature
* Native APIs for data access

After I receive products, I want to separate by type.  SK2 provides a property for the type as defined on the app store server.

VerificationResult => verified vs unverified.  Every time I receive the transaction, it passes through a verification process.  SK2 does transaction verification for you.  How you handle it is up to you.

Now I can use `checkVerified` on the result of the purchase.  

After the user has the content, I need to ensure I finish the transaction.

Sometimes the customer has to do extra verification on the account.  In these cases, the purchase result will be in a pending state.  

Enable 'ask to buy' in my xcode environment.  StoreKit configuration file, editor=>enable ask to buy.  

# Demo summary
* Request products
* Start a purchase
* Handle PurchaseResult
* Verify transaction
* Receive transaction updates

Can verify transactions yourself using crypto etc.

# Transaction history
1.  Access all transactions
2.  Latest transactions
3.  Current entitlements


## Current entitlements
* All proudcts the user is entitled to
* No revoked transactions
* No consumeables

## Transaction history
* All transactions are availble immediately upon app download
* Transactions automatically update on every device
* Real-time updates

Users won't need to restore purchases.  Everything should automatically be fetched by StoreKit.  

## Restoring purchases
* Immediately re-synchronizes all transactions
* Provide UI for users to initiate
* Should be very rare users need this.
* Always requires user authentication
* Only use in response to user input

## Transaction history
* All transactions are available in all original StoreKit APIs
* and vice versa
* Purchases with SK2 are available in the unified receipt.
* Subscription status

# Subscription status

* Latest transaction
* Renewal state => current state of the subscription
* Renewal info

Data can change without a transaction.  e.g. "auto-renew status" which tells you if the user has turned on/off.
auto-renew product ID => user downgrade, etc.
Expiration reason 

JWS.  SK2 will automatically validate it for you.

Note subscriptions return an array of statuses, because a user may have multiple subscriptions to the same product.  e.g., subscribed themselves, plus family sharing.  Use highest status in array.

What happens when I purchase a subscription product?  After I confirm the purhcase, status is displayed in the store.  Let my user know what they're subscribed to and when their subscription will renew.

# Signature validation
1.  header
	1.  algorithm => ECDSA
	2.  certificate => x5c.  Indicates entire chain is in the data.
2.  Payload.  Once you validated signature, you can read the data from here.
3.  Signature.  Generated from header and payload.

1.  Ensure bundleID matches your app
2.  Consider embedding this inside the app
3.  Perform device validation check.  

```swift
guard let localID = AppStore.deviceVerificationID?.uuidString.lowercased() else { fatalError() }
let combinedValue = trasnaction.deviceVerificationNonce.uuidString.lowercased() 
let hashedValue = SHA384(combinedValue)
if hashedValue == transaction.deviceVerification {
	// valid!
}
```

These are for IAP only.  For app receipt, use another API.  Also offering app store server APIs for these.

# Wrap up
* Powerful new APIs for in-app purchase
* New JWS transaction info
* Transaction detail and history API
* Additional subscription information

[[Manage in-app purchases on your server]]
[[Support customers and handle refunds]]

* https://developer.apple.com/documentation/appstoreserverapi
* https://developer.apple.com/storekit/
* https://tools.ietf.org/html/rfc7515
* https://developer.apple.com/design/human-interface-guidelines/in-app-purchase/overview/introduction/
* https://developer.apple.com/in-app-purchase/
* https://developer.apple.com/documentation/appstoreservernotifications
* https://developer.apple.com/design/human-interface-guidelines/subscriptions/overview/
* https://developer.apple.com/documentation/storekit


