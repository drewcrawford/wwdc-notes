# Feature overview
Family sharing has become an important part of the apple ecosystem, bringing content and services to devices etc.

Family Sharing is now available for IAP.  #storekit 

## Family sharing for IAP
* Auto-renewable subscriptions and non-consumable in-app purchases
* Share purchases with up to 5 family members
* Added to new or existing SKUs
* Easy to enable in ASC
* Improve retention KPIs

## Why family sharing?
Key metrics

* Subscriber growth
* Engagement
* Reduce churn
* => Increased CLV

## Customer benefits
* Share services with family
* Seamless OOB experiences
* Privacy friendly
* Customer value

## Getting started
One adult in the household can invite up to 5 family members to join.  Each member uses their own account.

Each family member gets instant access to group subscriptions and content eligible for sharing.

Settings app -> "Share new subscriptions" switch.  On by default.


## Auto-renewable subscriptions
New purchases -> enabled by default
Existing purhcases -> Purchasers must opt into sharing from Manage Subscriptions page.  To ensure acknowledging new terms.

Non-Consumables -> Subscriber sharing preferences.  This must be turned on (off by default?)

Non-consumables
New purchasers
Sharing is enabled if:
* Purchase sharing is enabled in iCloud
* The family is sharing purchases
* The app is not hidden from purchase history

Existing purchasers, restore purchase unlocks content if: see above.  Have to use "restore purchases" button.

Purchasers will receive a PN when family sharing becomes available on an existing subscription SKU.  PNs will be sent to customers that purchase a new family sharing SKU but have turned off the master toggle.

Either way, PN will link customers to the screen in settings.

## Best practice
* Consider including the functionality in your onboarding flow
* Offer family sharing as a separate subscription tier?
* If you choose to offer family sharing as a higher tier of surface, group the higher tier with individual tiers in ASC.
* Consider marketing the functionality.

## Business considerations
* Consider feature value prop
* Determine adding to a new or existing SKU
* Consult marketing teams to develop merchandising materials

# Engineering and implementation
"based in Cupertino"

## Enabling Family Sharing
Visit ASC, navigate to the subscription or nonconsumeable product.

**Enabling family sharing cannot be reverted.**

`SKProduct` -> `isFamilyShareable` boolean.
Dynamically control UI in your app.


## Entitling service for family members
Shared purchases are available to all family members.
You will see a transaction in the payment queue like a normal purchase.

In-app purchase best practices
* protocol should be initialized at launch `SKpaymentTransactionObserver`
	* transactions update: `SKPaymentQueue`
	* This ensures your app never misses a transaction
* Critical for Ask to Buy, etc., or any case where the transaction may be finalized outside your app
* Verify customer status before merchandising products
* Retrieve local app receipt and validate
* Determine if entitled, else merchandise appropriately


### ex
1.  Initiate purchase
2.  Transactions created for each family member.
3.  Your app will see this on launch or in realtime.
4.  We implemented a 1hr delay, to allow the purchaser time to disable sharing

### ex: existing subscription purchase

Not enabled by default.
1.  Purchaser enables from manage subscription page.
2.  Transactions created

### receipt
Ownership indicator
Field `in_app_ownership_type`
format: string
possible values: `FAMILY_SHARED` or `PURCHASED`
Allows you to distinguish between purchaser or family members.

Available with VerifyReceipt service or App Store Server Notifications

`responseBody.Latest_receipt_info`

## Handling subscriber events

Revoking Access for Family Members
* purchase disables sharing
* purchaser is refunded
* Family member is no longer in the family group


`didRevokeEntitlementsForProductIdentifiers` callback.  Signal that some entitlements may have changed.

App store server notification type: `REVOKE`.  Exclusive for family members

**It's critical to iterate through the entire receipt, may have multiple entitlements.**

 | Notification type         | Sent when                                                    | Purchaser | Family member |
|---------------------------|--------------------------------------------------------------|-----------|---------------|
| INITIAL_BUY               | A subscription is first purchased                            | Yes       | No            |
| PRICE_INCREASE_CONSENT    | When a subscriber is required to consent to a price increase | Yes       | N/A           |
| CANCEL                    | Auto-renewable subscription is refunded                      | Yes       | N/A           |
| REFUND                    | Transaction refund of IAP types                              | Yes       | N/A           |
| REVOKE                    | Refund or IAP is no longer shared with family member         | N/A       | Yes           |
| DID_FAIL_TO_RENEW         | sent at first failure attempt to take payment for renewal    | Yes       | Yes           |
| INTERACTIVE_RENEWAL       | Subscriber resubscribed or upgraded subscription             | Yes       | Later 2021    |
| DID_CHANGE_RENEWAL_PREF   | A customer downgrades/crossgrades                            | Yes       | Later 2021    |
| DID_CHANGE_RENEWAL_STATUS | A customer decides to voluntarily churn, or opt back in      | Yes       | Later 2021    |
| DID_RENEW                 | Find out when a subscriber renews each period                | Yes       | Later 2021    |
|                           |                                                              |           |               |

# Update in Sales and Trends
Subscription report
How many unique paying subscribers do I ahve by product?
How many customers have access including family members?

New column added with count of customers who have access to the subscription including primary and family.  Only populated if the record represents more than 3 active subscriptions

# Summary
Consider enabling FS for new/existing products
Clearly communicate FS support
supports cuostomer privacy and reduces account/password sharing
Follow best practices
* initialize observer at launch
* verify subscriber status before merchandising products