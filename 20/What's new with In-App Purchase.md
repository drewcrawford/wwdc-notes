
# Server-side

## Refunds
1.  Purchases 100 gems
2.  Contact apple for support
3.  Issue a refund
4.  Contacts you for support with remaining gems
5.  Check if payment has been cancelled??

### Refund management
Allows you to manage access to in-app content
Manage refund abuse
Resolve customer issues swiftly
Rebalance game economy

### Refund notification

Push approach
Notified upon status change
Similar architecture as subscriptions
Only receive new receipts when needed
Scalable

Notice that for consumeables, non-renewing subs, and non-consumeables, you'll get refund.  For renewing subs, you'll get cancel.

1.  Set up URL in app store connect
2.  ATS requirements
3.  Receive notifications

This is sent *after* we issue a refund.
We're not giving you any information about the customer, just about the purchase.

| Field name                | type   | location              | example value    | meaning                                                                                       |
|---------------------------|--------|-----------------------|------------------|-----------------------------------------------------------------------------------------------|
| `original_transaction_id` | String | `latest_receipt_info` | 1234567          | Unique identifier for the cancelled payment                                                   |
| `cancellation_date_ms`    | String | `latest_receipt_info` | 1559930400       | Date payment was cancelled                                                                    |
| `cancellation_reason`     | String | `latest_receipt_info` | 0 or 1           | Reason customer provided for refund, I guess it means did they allege a problem with your app |
| `bid`                     | String | Top level             | `com.my.app`     | Bundle identifier for the app                                                                 |
| `product_id`              | String | `latest_receipt_info` | `com.my.product` | Product that was cancelled     

Look for `password` in the payload, which is a shared secret for your app.

You're responsible for deciding which actions to take.  Promote a healthy community.

## Managing subscription
### Lifecycle
* Subscribe
* autorenew
* disable/enable autorenew
* upgrade/downgrade
* cancel

This timeline is quite complicated.

## app store server notifications

We notify you upon status changes
Only receive new receipts when needed.
Scalable

[[In-App Purchases and Using Server-to-Server Notifications - 19]]

### New `DID_RENEW` notification
Sent for each successful autorenew

## Best practices
* Rely on app store server notifications for updates, don't poll `verifyReceipt`.
* Use `verifyReceipt` for a few use cases
	* For recovery after an outage
	* Immediately determine entitlements for a customer

[[architecting for subscriptions]]

## Using verify receipt and notifications

Renewals are critical, do a double-check
* Enable app store server notifications to receive DID_RENEW
* Schedule one call to verify receipt at the expiration date for each subscription
* When you receive DID_RENEW, you can cancel
* But if there's a delay, you can fall back

## family sharing for IAP
Family sharing with IAP can improve retention

Note that once you turn on family sharing for a product, you can't turn it off.

Subscribers can turn off sharing later.
For nonconsumeables, all purchases will be shared with the family.

Not only are we creating a transaction for the purchaser, we're creating one for each family remember.
Each family member has unique receipt.
Keep track of the original transaction ID for each family member.
Existing non-consumeables require Restore Purchases for these to be available to all family members.

## Behavior across content types

|                     | Auto-renewing subscriptions                                    | Non-consumeables                                                                                                                                                  |
|---------------------|----------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| New purchases       | Family sharing is enabled by default                           | Sharing is enabled if: * Purchase sharing is enabled in iCloud * The family is sharing payment, and * The app is not hidden from purchase history                 |
| Existing purchasers | Customers must opt into sharing from Manage Subscriptions page | Restore purchases unlocks content if: * Purchase sharing is enabled in iCloud, * The family is sharing payment, and * The app is not hidden from purchase history |


# StoreKit

#storekit

## Family sharing
`.isFamilyShareable`


Under ordinary circumstances, receipients of the sharing get a "restore purchases" type transaction appearing in their queue.  So your existing "restore purchases" type logic should handle this, as if they're a normal customer.

However, the "disable sharing" case is a bit different, so we have a new API `paymentQueue(:didRevokeEntitlementsForProductIdentifiers:)`


* Purchaser disables family sharing
* Customer leaves family group
* Purchase is refunded


## IAP apple watch
#watchos 

Pretty much the same as any other platform.  Note that if you do local validation, use `WKInterfaceDevice` instead of `UIDevice`.


## Subscription price increase flow

When you want to raise prices on customers, you increase price in app store connect.

We are adding a new interstital when launching your app to handle this case.  You can disable this by implementing a delegate method.  New API to show the sheet at a time you prefer.

## SKOverlay

Unlike `SKStoreProductViewController` it's only used for apps, and the UI blends better with your app.

Transition users from your app clip to full app
But can also promote other apps
Customers can install directly from the overlay.

[[exploring app clips]]
[[Streamling your app clip experience]]


`AppClipConfiguration` - transition users from app clip to full app.  Can only display the full app for the current app clip.
`AppConfiguration` - display any app

Analytics tokens
Set/get arbitrary key values.  Integrate with `SKAddNetwork`
Can set `bottom` position or `bottomRaised`.

Delegate.

## SKAdNetwork

3 stakeholders
* ad networks
* source apps
* advertising apps

Can now call `updateConversionValue` to provide additional info about how well the user converted.  6-bit value defined by advertiser.
Can call this API multiple times.  Higher values overwrite lower values.  Values that are lower than current value are ignored.

Source apps must put ad network ID in their Info.plist
Advertising apps prepare the postback when the app first launches.

