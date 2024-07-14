Learn how to leverage your server to build great In-App Purchase experiences with the latest updates to the App Store Server API, App Store Server Notifications, and the open source App Store Server Library. After a recap of current APIs, we'll introduce updated endpoint functionality, new transaction fields, and a new notification type. We'll also discuss best practices for the purchase lifecycle, delivering content, and targeting offers, so you can become a server power user.

# Pillars of app monetization
* app store connect
* storekit
* app store server

Interact with the app store via a server-to-server api
obtain transaction information
extend renewal date, etc.

* transaction history apis
* refunds and customer sat apis
* app store server notification apis

these 12 endpoints replace and go beyond the verifyReceipt endpoint, deprecated in 20203.

# server notifications

Allow the app store to inform you of updates
Subscription lifecycle changes
Refund lifecycle changes

# server library
Foundation for using the app store server api and notifications.  Server library is designed to simplify your integration.

Provides a client for the app store server API, making it easier than ever to get started with these endpoints
Decodes signed app store server notifications
Extracts transaction info from deprecated receipts
creates promotional offer signatures
Available for production use!!

Java, Python, NodeJS, and Swift.  It's now the recommended way to use APIs in these languages.

Download the library via appropriate package manager for each language.

Open source.  We encourage you to submit feedback.

[[Meet the App Store Server Library]]

# Purchase lifecycle

variety of IAP types.

| type                        | repurchasable | renewable |
| --------------------------- | ------------- | --------- |
| non-consumable              | no            | no        |
| consumable                  | yes           | no        |
| non-renewing subscription   | yes           | no        |
| autorenewable subscriptions | yes           | yes       |
To learn more about purchase lifecycle, we'll do an example.

1.  Purchases consumable
2. Send signed transaction info to your server
3. app store sends ONE_TIME_CHARGE notification
4. Customer requests refund
5. app store server sends CONSUMPTION_REQUEST info to provide info about customer's use of the product.
6. You send consumption information
7. REFUND or REFUND_DECLINED

If no exception thrown, we successfully submitted, etc.

## new features

New notification type sent for purchases of
* consumables
* non-consumables
* non-renewing subscription
ONE_TIME_CHARGE.  Now when someone purchases these, we send the notification.  
Use with subscription notifications to track every in-app purchase type.

New notification available today in sandbox.
Available later this year in production.

If you provided an app account token, you can find it here.  Use this value to understand which customer account made the purchase.  Allows you to immediately unlock content for purchase.

No need to call server API or wait fo ra call from the device.

Refunds.  As mentioned, CONSUMPTION_REQUEST is a refund for a consumable in-app purchase.  

CONSUMPTION_REQUEST is now sent for autorenewable subscriptions
* still sent for consumables.
consumptionRequestReason indicates reason for refund request.  ex UNINTENDED_PURCHASE.


When you call sendConsumptionInformation, you provide info on customer's use of the product.  We want you to take a more active role in the decisioning process. You can now submit a preference for granting / denying the refund.  Use the new `consumptionRequestReason` field to inform your preference.

Your biggest role is to deliver a seamless purchase experience that doesn't generate refund requests in the first place.

# Delivering content

Best practices.

1.  Purchase in-app product
2. Send signed transaction info
3. Your server grants content.  ex update balance on server
4. Indicate content granted from server
5. App marks transaction as finished.  

Let's focus on content granting step.  Best practices.  Because you have sole control over your server, it should be your only source of truth. Track content delivery server-side.

Don't use finished status as source of truth.  Once your server has granted content, mark it finished as a signal to the app store server. Don't use finished status as source of truth.

Validate transaction signatures regardless of source.  Before granting content.

Important to discover new/updated transactions quickly.  A variety of ways.

* New or updated transactions from device
* Enable app store server notifications v2, especially for renewals.  Don't rely on the device
* Set `appAccountToken` on purchase.  When you receive server notifications, link the transaction data to the customer without requiring the device.
* Use Get Transaction History endpoint if you think you missed a purchase.

Can request from a specific revision if we have one.  

## update
New version of get transaction history
Fetch full history of all customer's purchases in your app
* all product types
* refund status
* finished status

New use cases
* display full purchase history
* Refresh purchase entries
* Audit consumable balances


Returns all the same data as before and more.
Original version is deprecated!  Migrating is simple because new endpoint is substantially similar to the original.  Update the version in URL is v2.  Prepare to receive finished consumables, and you're ready to go.



# Subscriptions and offers

Several different payment modes.  
* free trial?  Great way to encourage customers to give your services a try.
* First renewal period, e.g. intro offer
* Pay up front offer.  Prepay a period at a reduced price.


| offer type            | new subscribers | existing subscribers | churched subscribers | distribution          | eligibility          |
| --------------------- | --------------- | -------------------- | -------------------- | --------------------- | -------------------- |
| introductory offers   | yes             | no                   | no                   | automatic             | automatic            |
| offer codes           | yes             | yes                  | yes                  | developer to customer | developer-determined |
| promotional offers    | no              | yes                  | yes                  | developer to device   | developer determined |
| Win-back offers (new) | no              | no                   | yes                  | automatic             | automatic            |

[[Implement App Store Offers]]

Promotional offer workflow.

1.  Customer disables auto-renew
2. DID_CHANGE_RENWEAL_STATUS:AUTO_RENEW_DISABLED
3. You create promotional offer signature
4. Customer buys with offer
5. SUBSCRIBED or OFFER_REDEEMED

But how do you create that signature?  app store library again.

## updates

What types of offers to create?  Who should receive them?  We want to give you more information.

price: includes offers applied
currency

offerDiscountType
* FREE_TRIAL
* PAY_UP_FRONT
* PAY_AS_YOU_GO

Renewal info for each of your subscriptions.  Understand what will happen the next time you autorenew.  3 new fields

* renewalPrice
	* Reflects expected offer applied, if any
* currency
* offerDiscountType
* For expected offer applied, if any

Renewal info now has the price / offerDiscountType on a forward looking basis.  

# Review

* ONE_TIME_CHARGE notification
* Consumption information updates
	* Auto-renewable subscriptions support
	* Consumption request reason field
	* Refund preference support
* Get transaction history now includes all purchases
* New fields for offers, pricing, and subscriptions


# Wrap up
* submit feedback through feedback assistant
* check out the app store server library
* submit issues and PRs on GitHub
[[What's new in App Store Server APIs]]
[[Explore in-app purchase integration and migration]]

# Resources
* https://developer.apple.com/documentation/appstoreserverapi
* https://developer.apple.com/documentation/appstoreservernotifications
* https://developer.apple.com/documentation/appstoreservernotifications/consumptionrequestreason
* https://developer.apple.com/documentation/appstoreserverapi/currency
* https://developer.apple.com/documentation/appstoreserverapi/get_transaction_history
* https://developer.apple.com/documentation/appstoreserverapi/offerdiscounttype
* https://developer.apple.com/documentation/appstoreserverapi/price
* https://developer.apple.com/documentation/appstoreserverapi/refundpreference
* https://developer.apple.com/documentation/appstoreserverapi/renewalprice
* https://developer.apple.com/documentation/appstoreserverapi/send_consumption_information
* https://developer.apple.com/documentation/appstoreserverapi/simplifying_your_implementation_by_using_the_app_store_server_library
* http://feedbackassistant.apple.com/
