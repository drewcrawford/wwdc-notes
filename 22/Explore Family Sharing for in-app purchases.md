Family Sharing for in-app purchases lets people share their auto-renewable subscriptions and non-consumables with up to five additional family members, helping you attract new subscribers, increase user engagement, and improve retention. We'll review how you can enable this feature in App Store Connect, provide best practices when using the feature with StoreKit and App Store Server Notifications, and help you deliver a great in-app purchase experience.

# What is family sharing for IAP?

**Family sharing is available for in-app purchases**

* auto-renewable subcriptions and non-consumable IAPs
* share purchases with up to 5 family members
* added to new or existing SKUs
* easy to enable in ASC
* Improve retention KPIs
# Why family sharing?

## Key metrics
* subscriber growth
* engagement
* reduce churn
* increased CLV

Once you've acquired michael as a subscriber, he can extend his subscription to the rest of his family members, multiplying customer touchpoints to service/app.  With the whole family now engaging with your product... keep them coming back.

Share services with family.  Seamless out-of-box experience.
Privacy friendly
Customer value.


# Making the most of FS

Customers must first create a family group in order to begin.  Once adult per household can do this, inviting up to 5 of their family members to join using apple IDs.  Each member uses own account, experience stays personalized and private.

Instant access to the group's subscription and content.

settings app 'share with family'.  Turned on by default, but customers can choose to opt out at any time.  Selecting any of the listed apps let you edit purchases and choose if on/off for specific sub.  

Existing subscriptions -> purchaser must opt in via setting.  Customer needs to fully acknowledge updates to subscription terms.

We encourage you to message customers.  But we may do so directly.

1.  purchasers receive a PN when family sharing becomes available on existing SKU
2. or to new family-sharing enabled sku, but have turned off the main sharing toggle

Non-consumables

automatically enabled for both new/existing as long as purchase sharing is enabled in icloud, family is sharing purchases, and app is not hidden from purchase history.

may have to hit restore purchase button.

## Best practice
Consider including the functionality in onboarding flow.
Possible to offer family sharing as a separate subscription tier.  Remember to group higher tier with individual tier/tiers in ASC.  This allows you to offer upgrade messaging.

## Business considerations
* consider feature value proposition
* Determine adding to a new or existing SKU
* Consult marketing teams to develop merchandising materials

# Engineering and implementation

## Enabling family sharing

via ASC.
**Enabling family sharing cannot be reverted**
Customers may have based their purchasing decision on this functionality.

SK will now return `isFamilyShareable`.  leverage this value when merchandising family shareable products.

Available from iOS 14+.  

## Entitling service for family members

Shared purchases are available to all family members.  Application will likely already handle the family transactions without making changes.  Client side, server side.  Automatically purchases are available to all family members.

Transactions behave the same way for family members as they do for purchasers.
Each family member has their own transactions and app receipt

## best practices
* Listen for transaction at launch and lifecycle
* Critical for ask to buy, PSD2, and offer code transactions
* Verify customer status before merchandising products. Check via sk or app receipt.  Determine if entitled, else merchandise appropriately.

[[Implement proactive in-app purchase restore]]

## ex
1.  Initiate purchase
2. Transaction created for each family member
3. application will see new transaction upon launch or in realtime.
4. Validate and entitle service like any other transaction.
5. We implemented a delay for family transaction.  Allows purchaser time to disable family sharing if they choose to do so.

existing subscription purchase.
1.  Not enabled by default
2. Enable subscription
3. Transactions are created shortly after enable

Ownership indicator can be `familyShared` or `purchased`.  To have custom onboarding experience for family members, this is the property to use.  Or this could be helpful if you wanted to identify a customer with the ability to manage the subscription.  Look for the value `purchased`.  On device, with v2 service or app store service notifications.
## Handling subscriber events

Unique scenarios to be aware of.  Times you may need to revoke access from a family member.  Such as, when a purchaser disables sharing.
Purchaser is refunded.
Family member is no longer in the family group.

Check for `revocationDate`.  Note that other transactions may entitle customers to the same or different products.  Check and make sure the customer is receiving what they are paying for.

original storekit - need to use a callback method called `didRevokeEntitlementsForProductIdentifiers`.  

app store server notifications type: REVOKE.  Subscriber event: family member lost access
notification available for v1 and v2 notifications.  When you receive either of these callbacks, re-establish the correct entitlements.  Important ot iterate through the entire transaction history as customers may have had multiple entitlements.  If one is revoked, may be another transaction!

Ensure there is no service interruption.

Note that family member do not get REFUND, REFUND_DECLINED, or PRICE_INCREASE, those only go to purchaser.  Otherwise you get them all.

If you have not already added your server endpoint, do so at any time.

# Sales and trends
How many unique paying subscribers do I have by subscription?
How many customers have access per subscription including family members?

Subscription report has a 'subscriber' column.  Provide you the count of customer who have access to the subscription.  Including purchaser and family members.   Only populated if it represents more than 3 active subscriptions for your product.

# Summary

* consider enabling FS for new or existing products
* Clearly communicate FS support when merchandising to customers
* Family sharing supports customer privacy and reduces account and credential sharing.


# Resources

* https://developer.apple.com/documentation/appstoreservernotifications
* https://developer.apple.com/app-store/subscriptions/#family-sharing
* https://developer.apple.com/in-app-purchase/
* https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase/supporting_family_sharing_in_your_app
* https://help.apple.com/app-store-connect/#/dev45b03fab9
