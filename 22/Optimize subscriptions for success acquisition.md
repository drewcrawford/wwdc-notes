Learn how you can acquire subscribers and grow your business using App Store features. We'll explore subscriber acquisition strategies, share implementation best practices, and show you how to integrate these processes into your app for success.

# Acquisition offers
650M weekly visitors.
1 billion+ paid subscriptions across services

introductory offers. Standard intro price for new subscribers.  ex 1 week free.
Offer codes take the functionality of intro offers but more flexibility.  Custom code unlocks a special offer inside the app.

60% of active paid subscriptions started with an intro offer.

3 discount types.
* free trial -> initial price for period of time.
* pay up front -> one-time discounted purchase before renewing at regular rate
* payg -> multiple renewals at discounted price.

Offers should always be utilized as part of great customer UX.  Don't be a scam.

paid offers are popular in markets like greater china.  Paid offers can also be used for content licensing issues or strong brand considerations.

common free trial lengths
* monthy -> entertainment, social, books, music
* weekly -> health and fitness, productivity, education, weather
* 3-day -> photo and video, games, utilities, business


monthly -> five eyes
weekly -> greater china, latam
3-day -> SEA, pan EMIEA
# Sample strategies

## Onboarding
### Content sampling
Scenario: keep new or prospective subscribers engaged with free content

## personalized messaging
offer codes
understand customer motivations and behavior to serve relevant offers

## subscription plans
Scenario: offer longer term subscription plans up front to strengthen customer retention.
Across the platform, we observe that yearly and halfyearly SKUs are most popular offerings.

See ocean journal subscription options.  Yearly, monthly term.  Provide users with a 7-day free trial.  33% annual discount on their yearly sku, compared to monthly.

Discounted offering is of benefit to ocean journal, as it helps retain customers for long periods of time.

KPIs
* conversion rate
* retention rate
* subscription activations

## Marketing

### Offer codes
Scenario: use on and off-line marketing channels

ability to surface offers before a user even downloads your app.  Redemption flow will prompt users to install if not installed.  

Utilize paid media channels to drive customers into sub funnel.

Recommend testing/tailoring campain to ensure it's relevant for your audience.  To distribute offers through offline media channels, use compelling offers thorugh events or out of home.

## Member referrals
Can be executed using offer codes.  Allow current customers to refer friends, etc.

KPIs
* offer codes redemptions
* Offer codes conversion rate

# Engineering for success

## intro offers
Need to consider for introductory offer is that a customer may have previously redeemed the offer and no longer eligible.  Ensure merchandising is in sync with app store payment sheet.

* check customers eligibility before merchandising.  `isEligibleForIntroOffer`.  
* server side, can use JWSTransactions `offerType=1`
* latest receipt: `is_trial_period` `is_in_intro_offer_period`.

Merchandise offer details using
* SK2: `introductoryOffer`
* SK1: `introductoryPrice`

leverage storekit because this may be dynamically available, don't hardcode

Use Reset Eligibility feature to re-test an introductory offer using the same Sandbox Apple ID.

`showManageSubscription(in:)`.

[[Introducing StoreKit testing in Xcode]]

## Offer codes
* multiple offers
* use existing IAP SKU

can embed offer code link.  Presented with app store offer redemption sheet.
Customer presented with offer redemption message.  Ensure that your app handles external transactions gracefully.

SK2: Transaction.updates
SK1: `SKPaymentTransactionObserver`.
Call `finish()` after purchase was delivered.  Unfinished transactions remain in the queue.

Offer codes testing in xcode
`presentCodeRedemptionSheet()`
Locally configure offer codes

Test successful redemption and app handling of new transactions.  On backend systems, easily identify offer code redemptions, containing offerType = 3, and offerIdentifier.
Receive real-time notification for offer redemption
type: `OFFER_REDEEMED`

[[Get started with custom offer codes]]

Redemptions aggregated by territory.  Specific custom codes, for one-time codes it's blank for customer privacy.

Visit our auto-renewable subscription documentation.  Resources and appliable links.  Thanks
