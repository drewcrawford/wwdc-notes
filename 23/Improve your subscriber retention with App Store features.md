Learn how to minimize churn and win back subscribers on the App Store. Explore App Store data, review different types of subscriber churn, discover tools you can use to enhance your retention efforts, and learn implementation best practices.

 Autorenewable subscriptiosn have proven to be a popular way to grow businesses on the app store, etc.
Since autorenewable subscriptions are a recurring revenue model, tyhey have a unique customer lifecycle.

1.  Discovery
2. Acquisition
3. Engagement
4. Retention

Leaky bucket.  Plug the holes, retain more subscribers.

85% proceeds in year 2

Churn:
* voluntary.  ex, you don't have a good product basically.
* involuntary ex., billing issue

compared to US, churn rates are higher in mainland china.

lower side: health and fitness, music.
higher: entertainment, lifecycle

involuntary churn is not necessarily linked to region and category.

From 85% monthly retention to 95% monthly retention rate, it's 3x more revenue.

# Reporting
* retention rate
* subscription report
* subscription events report
* retention benchmarks (app analytics)

# Features and sample strategies

## features
automatic:
* billing retry
* billing problem message
choose to implement
* billing grace period
* subscription offers
## billing retry
* attempts to recover failed renewals
* 60 day recovery window
* continue accruing days of paid service
## billing problem message

If they try to use your service, we push a VC to prompt them I guess

## billing grace period
* save customers from involuntary churning
* prevent lost revenue
* provide customer interrupted service
* retain original customer renewal date

40% of customers correct their billing information within 3 days  of billing period being recognized.  75% within 16 days, 90% within 28 days.

80M subscriptions recovered during billing grace period.

## subscription offers

free or discounted subscription for specific duration distributed inside of your app.

Offer codes - additional flexibility for merchandising offers inside and outside your app.

Use offers to save low-engaging customers.
Can try to re-engage customers that already turned off autorenewal.  Remaining days?  Already churned.


for low engagement
* communicate value of service
* provide a free period
* offer to upgrade or downgrade to a different tier of service
* distribution: email, push notification

turned off autorenew, still has days remaining
* reach customers before they lose service access
* showcase the benefits of your sub
* provide a a personalized discount offer
* distribution - in app

fully churned
* attempt to win back
* determine optimal time to begin providing discounts
* provide deeper discounts as length of churn increases
* distribution: email, push


# Engineering for success

App store checks for billing issues.  
* valid active payment method on file
* Contact customer to update payment method
* Conduct fraud and risk check
* Send subscription renewal reminder

## billing retry
automatically entered.  Apple attempts to recover the subscription.  Sub status may change.

Use SK property `inBillingRetryPeriod`.  ASN.  Notification type `DID_FAIL_TO_RENEW`.

renewal state updated to `subscribed`.  ASN `DID_RENEW:BILLING_RECOVERY`.  

When we reach the limit, `expired` ASN `EXPIRED:BILLING_RETRY`.

### best practices
* don't merchandise to customers whose subscription is in billing retry.
* note that we include billing retry in 'days of paid service' e.g. to get to 85%.
*  can opt out by disabling the subscription
* Leverage properties, such as `expirationReason` and ASN: `expirationIntent`.

Use cases to validate
* entering retry
* Recovery from retry
* exiting retry
* cancelling

## billing problem message
* defer message on uninterruptible views
* Use SK message api
	* property: `.reason`
	* value: `billingIssue`

[[What's new in StoreKit 2 and StoreKit Testing in Xcode]]
[[Explore testing in in-app purchases]]

## billing grace period

Enable via ASC.

additional scenarios.  `inGracePeriod` ASC: `DID_FAIL_TO_RENEW:GRACE_PERIOD`.

recover: `subscribed` aka `DID_RENEW:BILLING_RECOVERY`
back to retry: `inBillingRetryPeriod` aka `GRACE_PERIOD_EXPIRED`.

additional usecases
* entering grace period
* recovering in grace period
* exiting grace period
* cancelling in grace period

## subscription offers

* identify subscriptions in cohort
StoreKIt `RenewalSTate = subscribed` and `willAutorenew = false`

subscriptpion status api: status = 1, autorenew = 0

`DID_CHANGE_RENEWAL_STATUS:AUTORENEWAL_DISABLED`

### best practices
* use `promotionalOffers` property of StoreKit.  
* Send the signature request to the server when a customer is ready to purchase
* use app store server library
* identify offer redemption using
	* offerType = 2, and offerIdentifier.
* receive app store server notifications for offer redemption.  Type: `OFFER_REDEEMED`.

[[Meet the App Store Server Library]]
[[Subscription Offers Best Practices - 19]]

Auto-renew off: fully churned
RenewalState = expired, willAutoRenew = false
status = 2, autoRenewStatus = 0
ASC: EXPIRED:VOLUNTARY

### best practices
merchandise offer in-app or outside
Use deep link to embed the offer code

[[Subscription Offer Codes]]
[[Optimize subscriptions for success acquisition]]

Leverage properties
* new transaction properties (price, currency, offerDiscountType)
* recentSubscriptionStartDate (ex, customer loyalty).

