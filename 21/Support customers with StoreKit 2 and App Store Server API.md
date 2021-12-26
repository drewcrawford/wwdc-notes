#storekit #appstore 
How SK2 can improve IAP, customer support, and handling refunds.

# What's new
* New framework: StoreKit 2
* showManageSubscriptions
* beginRefundRequest
* isEligibleForIntroOffer

App store server notifications
* Version 2

App store server API
* IN-app purchase history
* subscription status
* lookup order ID
* refund lookup
* renewal date extension
* consumption information request

Frictionless billing and authentication
1.5 billion devices
175 storefronts in over 40 languages
expanding price tier to over 500 (2022)
45 currencies across 195 payment methods
stacked payment methods

## Benefits
* Transactions are automatiacally synced across users devices
* Supports on-device purchase validation
* Secure JWS signed transactions
* Entitlement status
* `appAccountToken` - associate account with purchase

standalone features
* showManageSubscriptions
* beginRefundRequest
* Introductory offer check
* `isEligibleForIntroOffer` - eligibility check
* Product availabilitiy: `Product.SubscriptionOffer`.  Don't hardcode, leverage storekit.

## Subscription status
* Provides subscription state
* Returns only the current transaction.  Don't need to parse through history
* Maintain status for multi-platform services

Best practices:
* store original_transaction_id
* Use original_transaction_id to follow subscription lifecycle
* Use only server to server
* Use SK2 to check from app

## In-app purchase history
Retrieve customers' transaction history
Secure JWS signed transactions
Complements `beginRefundRequest`.
Check for updated or new transactions

Best practices
 * Handle paginated response, record `revision` token
 * Quickly retrieve latest transactions using last `revision` token
 * Call `finishTransaction` after products have been delivered to customers
 * Leverage App Store Server Notofications Version 2 for refunds and revoked transactions

## App Store Server Notifications

v2 benefits
* 28 unique events
* Single notification per event
* Supports family sharing
* Retry 5 times
* Available in sandbox

Best practices
* Respond with HTTP 200 RC when received
* SignedDate indicates when originally sent
* Validate notification signature for authenticity
* Provide unique server url for production and sandbox notifications
* Process notifications in real time

## Sandbox
Slow down or accelerate the subscription rewnewal rate per Sandbox Apple ID
Use a single sandbox apple ID across any storefront
Ability to clear purchases from sandbox apple id's purchase history

# Implementation approaches

* On-device
* Server to server

## Ondevice
* Products
* Verify transactions
	* verificationresult
* purchase result

* Transaction update
* currentEntitlement => grant products to customers
* Product.SubscriptionInfo => maintain subscription information (not entitlement information)

## Server-to-server
Customers expect to access services across platforms.

Previously,
1.  Sends base64 app receipt to your server
2.  send to appstore
3.  appstore responds
4.  etc.

App store server has no dependency on adopting storekit2.   Just use `original_transaction_id` instead of base64 app receipt.  `original?.transactionIdentifier ?? trnasaction.transactionIdentifier` .

With these changes, you no longer have entitlement logic to maintain on your server.  If your app supports older versions of iOS, this gives you a path.

## States
Active
Expired
Billing retry period ("through various customer communciations")
In grace period.  Only applies if you've opted in in ASC.  
Revoked.  (from family sharing)

## Server-to-server implementation
Lots of events.  Some may or may not apply.

Adoption phases
* Getting started
	* Enable version 2 in ASC
	* Separate URL for sandbox vs production
	* Prioritize key customer events
		* REFUND
		* Autorenewable (SUBSCRIBEd, DID_RENEW, DID_FAIL_TO_RENEW, EXPIRED)

Remaining events
DID_CHANGE_RENEWAL_STATUS
DID_CHANGE_RENEWAL_PREF 

Feature dependent
GRACE_PERIOD_EXPIRED
RENEWAL_EXTENDED
REVOKE
OFFER_REDEEMED
PRICE_INCREASE
REFUND_DECLINED
CONSUMPTION_REQUEST

Action on insights phase
Pro-active opportunities.  Proactive messaging, etc.

Ex, DID_CHANGE_RENEWAL_STATUS.  Customer disables auto-renew.  

# Customer support
## Order lookup
Today,c ustomers get receipts.  Now when a customer contacts you, your support team can ask them for the order ID on the receipt to look up IAP for the receipt using the order ID.
* Validate receipt
* Identify issues with the purchase
	* revocationDate
	* revocationReason
* Store the original transaction id
* Associate purchases (e.g. customer contacts)

## Refund lookup
Lookup a customer's refunded transaction
Identify refund history
Works across IAP types
Monitor suspicious activity
Respond to content delivery issues

### Best practices
Lokup a customer's refunded transactions
Use any original_transaction_id
Troubleshoot refund issues
## Renewal date extension
Extend renewal date for a paid active subscription.  Give customers free service.
handle temporary outage
Provide appeasement
Up to 2 extensions per year
Up to 90 days per extension
Does not count toward 1 year of paid service

### Best practices
* Store original_transaction_id
* Identify extension period
* Review extension eligilbity
* Align messages with business and marketing
## Subscription offer codes
* One-time discount code
* compensate for service issues
* Suggest alternative subscription options
* Customers can redeem on app store, or within the app `presentCodeRedemptionSheet`.

### Best practices
Provide redeem code inside the app
Customize in-app messaging
Integrate with existing channels - phone, email, web, in-app
Custom offer code redemption URL with code prepopulated
Redemption occursin app store, requires iOS 14 and iPadOS 14

Note subscriber never sees the code in the email flow.

## showManageSubscriptions
Provide subscription management functionality within app
Provide upgrade/downgrade/cancel flow
Provide alternative offers
Present save offer
exit survey

### Best practices
Present consistent experience
Provide contextual messaging
Review in-app support journey

## Benefits
* Improved customer experience
* Increased retention
* Higher engagement
* Positive ratings and reviews

# Handling refunds
* beginRefundRequest
* Send Consumption Information

Customer can request refund within your app
* In -app purchase assistance to customers
* Potential issues with IAP
* Notifications:
	* REFUND
	* REFUND_DECLINED
* Must be on iOS 15 or above

## Best practices
* Store `original_transaction_id`
* Build in-app messaging
* Flexible display of past purchases

## Send consumption request
"Refund decisioning system"
Influence refund decision by sharing information.
Key fields:
* Consumption: consumed in-app purchase
* Delivery: inform if an IAP was not delivered
* appAccountToken: identify reseller markets

## Best practices
Store data points for consumption fields for a minimum of 30 days
Store `original_transaction_id`
Respond within 12 hours
Feel free to send an updated payload if any information changed
Ensure customer consent is obtained
Not all fields are required

# Checklist
* enable server notifications version 2
	* sandbox first
* Integrate with existing support channels (support)
	* phone, email, web, in-app
* Leverage new SK2 features
	* beginRefundREquest
	* isEligibleForIntroOffer
	* showManageSubscriptions
* Sandbox testing features
	* storefrong chagne, clear purchase history, refunds, change renewal rate

