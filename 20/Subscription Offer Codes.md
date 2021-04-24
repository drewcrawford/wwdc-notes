# Offer Codes Introduction
Now available.

* For new, existing, and lapsed customers
* Unique one-timeuse alphanumeric code
* Flexible distribution for both online and offline
* Seamless customer redemption experience
* Subscription auto-renews at standard price

## Using offer codes
* Digital marketing channels (email)
* Alongside physical goods or services
* Physical retail or events
* Third-party partnerships (comarketing)
* Customer service

## Configuration and limits
* ability to  configure for new, existing, lapsed subscribers
* Up to 10 active offers per subscription SKU
* Create batches of 500 to 25k codes
* Max of 150k per app, per quarter
* Codes expire maximum of 6 months from creation

Can redeem in app store or in your app.  User will be prompted to download.

Code can be part of the URL.

We recommend communicating the expiration date and other availability requirements.

## Offer types

|                   | Introductory offers                      | Promotional offers                    | Offer codes                                                |
|-------------------|------------------------------------------|---------------------------------------|------------------------------------------------------------|
| Primary Use       | Acquiring new subscribers                | Retaining and winning back            | Acquiring, retaining, and winning back                     |
| Distribution      | In-app                                   | In-app                                | Online and offline marketing channels                      |
| Limits            | 1 per subscriber, per territory          | 10 active offers per subscription     | 10 active offers per subscription (150k codes per quarter) |
| Redemption Limits | 1 per subscriber, per subscription group | Unlimited, based on eligibility logic | 1 code per active offer                                    |

# Engineering Offer Codes

## Configuring Offer Codes
Durations.
PAYG, Pay up front, free trial

Offers can restrict redemption to these subscriber cohorts
* new
* existing
* expired

Use your *existing* subscription group and SKUs.

## Intro offer + offer code combos
Can support Intro offer + offer codes.  If allowed, intro goes into effect, followed by offer code.  Or if not, offer code begins immediately, but they will retain their eligibility for the introductory offer.

Only applies to subscribers that are eligible for the intro offer.  Subscribers that have already received intro offer, will not receiver another
New or expired subscriber to the subscription group.

Note: This configuration cannot be reverted.
Configuration does not change the subscribers Intro Offer eligibility, just the behavior at time of redemption.

## Redemption
### Restrictions
Subscribers are limited to one redemption per Offer Code
Each OC is uniquely ided by the name setup in ASC
Downgrades are not supported, SKU must be in same or higher SKU

### specifics
subscribers can redeem directly in app store.  Need to monitor payment queue on first launch.
app installed with redemption

in-app redemption sheet - redeem directly in app with `PresentCodeRemptionSheet()` API.  Requires iOS 14 and iPadOS 14.

URL support.  https://apps.apple.com/redeem?ctx=offercodes&id=######&code=XXXXXX
* ID - applicaton ID (static)
* code = offer code (dynamic)

This generally goes through appstore and is an external transaction

### external transaction
Offer Code redemption (can) occur outside of app
* intitialize at launch: `SKPaymentTransactionObserver`
* Ensures interrupted transactions and external transactions are not missed
* Critical for Ask to Buy, PSD2, and Offer Code transactions
* Call `finishTransaction` when purchase was delivered

## Subscriber Status
* standard monthly subscription
* introductory offer - 1 week free trial
* offer code - 1 week free

### don't allow
In the case we don't allow interop, 

1.  OC goes into effect immediately.
2.  Subscription renews at regular price.
3.  Subscription expires
4.  Subscriber now eligible for IO.

### allow

1.  Intro offer goes into effect
2.  offer code goes into effect
3.  subscription renews at regular price

### getting status
#### app receipt
Field: `offer_code_ref_name`
Reference name configured in ASC

Available with VerifyReceipt service
#### server notifications
Field: `offer_code_ref_name`
Available in `unified_receipt.Latest_receipt_info`

Notification types
* `INITIAL_BUY` (new subscribers)
* `INTERACTIVE_RENEWAL` (existing subscribers, like resubscribe or upgrade)
* `DID_CHANGE_RENEWAL_STATUS` (expected to churn)
* subscription autorenew re-enabled

# Setting Up Offer Codes in App Store Connect
## Configure offers
Select who is eligible (new/existing/expired)
Configure if we combo with intro offers or not
Territories
PAYG, PUF, free
Duration, currency, price

## Generate codes
150k new codes per quarter.  your app must be in ready-for-sale for user to redeem codes.

500-25k codes per generaton operation.
Expiration date: 1 day - 6 months.

Link template appears in ASC as well.

Deactivating codes will make them non-redeemable.  Subscribers who have already redeemed will not be affected.

You can have up to 10 active *offers* (not offer *codes* or code "batches" I guess? apparently) per subscription SKU.  If you need more than 10, consider deactivating codes that have expired or you don't need.

## distribute codes
?

macOS not supported.
# Measuring Offer Code Performance
New reporting in Sales and Trends
* how many paid subscribers acquried by offer codes
* Offer code to standard price conversion rate
* Top performing offer codes
* Various reports have references to offercodes now

# Summary
* consider supporting offer redemption within your app
* Only generate codes you plan to use (you can't recover unredeemed codes), and remember codes expire 6 monthsfrom the date of creation
* Clearly communicate offer details, eligibility, and redemption steps in your communications.