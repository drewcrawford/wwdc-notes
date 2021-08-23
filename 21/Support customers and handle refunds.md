#storekit 

[[Meet StoreKit 2]]
[[Manage in-app purchases on your server]]

# Support customers
## Benefits
* Mange your existing relationships with customers
* Increase retention
* Improve customer sat
* Grow revenue

## Channels
* Self-service
* Customer support

Alternatively they may contact you via social media, etc.  

1.  How to identify IAP?
2.  How to provide compensation?
3.  How to manage subscription?

## How do I identify in-app purchases?
Invoice via email.  Invoice order ID.

New `/inApps/v1/lookup/{customer_order_id}`

* Lookup IAP for invoice
* Validate invoice
* Identify issues with the purchase
* Refunded purchases

1.  Ask customer for order ID
2.  You look up
3.  Appstore returns status + JWS
4.  Provide support

status:
* 0 => valid order
* 1 => order ID is invalid
* 2 => no transactions were found

Link invoice order ID in your database for in-app purchases when they contact you about an issue.  

How do you look up past refunds?

`/inApps/v1/refund/lookup/{original_transaction_id}`
* Lookup for refunded transactions for customer
* Purchased within your app
* handle outage or scheduled maintenance
* Identify customer's refund history
* Across all content types

Can now include `refundDate` in your database.

## How do I compensate customers?

* In-app compensation?
* Discount on next renewal?

Subscription offer codes (iOS 14)
* One-time discount code
* Offer codes to compesnate for service issue.
* Suggest alternative subscription to customer
* Customers can redeem
	* on the app store
	* Within app - `presentCodeRedemptionSheet`

### Renewal extension API
* Extend renwal date for paid active subscription
* Offer free service for additional time
* Appeasement for temporary outage
* Up to 2 extensions per year for a customer's subscription
* Extend up to 90 days per extension
* Does not count toward 1 yr for 85%

`/subscription/extend/{original_transaction_id}`

ex,
1.  Customer contacts you
2.  You extend subscription

ex,
1.  Event was cancelled
2.  Just proactively extend

## Manage subscriptions?
`showManageSubscriptions()`
Display manage subscriptions page
Provide subscription management within app
Optionally present save offer and/or exit survey

Test manage subscriptions in sandbox

Can view, upgrade, downgrade, or cancel subscription.  Your server will receive app store server notification.

## Request a refund?
`beginRefundRequest(for transactionId:)`
* Customer can request refund within your app
* App will be notified
* App store will send server notification
	* REFUND
	* REFUND_DECLINED
* Test refund request in sandbox

## Summary
* Increase retention
* Improves customer satisfaction
* Higher engagement
* Positive ratings and reviews


# Handle refunds
Last year, we announced a new REFUND notification.

## Best practices
 * Find response strategy that works best for you
	 * e.g. deduct currency, revoke subscription service
 * Consider impact on game design
 * Use marketing and promotional tools
 * Provide clear messaging across communicaton channels

ex
1.  Purchase 100 coins
2.  Spend 100 coins
3.  Requests refund
4.  Issue refund
5.  You are notified
6.  Customer is notified
7.  Usually within 48 hour window

At a high level, each refund request goes through a decisioning system.  Includes transaction and other factors like their history.

Consumption API
`/inApps/v1/trasnactions/consumption/{original_transaction_id}`

When a customer requests a refund for a consumable, app store sends you new notification `CONSUMPTION_REQUEST`.  In most cases, customers start consuming content and knowing this information is helpful in our process.

Send information to the app store within 12 hours so it can be used to inform the decision.

PUT /transactions/consumption/{original_transaction_id}

* customerConsented => true, if the user has consented to sending
* consumptionStatus => In-app purchase was consumed (partially/fully/not at all).
* consumptionPlatform => app is cross-platform and where it was consumed
* sampleContentProvided => true if provided prior to purchase
* deliveryStatus => was it successfully delivered?
* appAccountToken => UUID associated with app's user account that you create.
* accountTenure => tenure of the account
* playTime => time played
* lifetimeDollarsRefunded
* etc

3 related notifications
* CONSUMPTION_REQUEST
* REFUND
* REFUND_DECLINED

## Shared benefits
* More transparency
* Improve refund process
* Better outcomes for customers
* Increased communication

# Wrap up
* Add custom help UI in your app
* Review customer support journeys
* Setup your server to receive notifications
* Take action after refunds
* Respond to consumption requests from the App Store

[[Meet StoreKit 2]]
[[Manage in-app purchases on your server]]

* https://developer.apple.com/documentation/appstoreserverapi
* https://developer.apple.com/design/human-interface-guidelines/in-app-purchase/overview/introduction/
* https://developer.apple.com/in-app-purchase/
* https://developer.apple.com/documentation/appstoreservernotifications
* https://developer.apple.com/app-store/subscriptions/
* https://developer.apple.com/documentation/storekit

