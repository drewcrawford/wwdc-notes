Discover how you can complement existing offer codes campaigns with custom, repeatable codes to provide even more flexibility to acquire and retain subscribers. We'll take you through the latest enhancements to offer codes, provide engineering guidance, explore best practices, and show you how to create new codes for your subscriptions.

# Offer codes overview

* flexible distribution both online and offline
* for new, existing, and lapsed customers
* seamless customer redemption experience
* subscription auto-renews

complementary to intro offers and promotional offers, but give you added flexibilty.  Distributed outside of your app.

Launched in 2020
unique, alphanumeric code
One-time use
redeemed in-app or app store
restricted code access

# Custom codes
Memorable custom code, such as NEWYEAR, to be redeemd by multiple customer
support mass distribution
redemption limits and expiration
simpler distribution and redemption flow

## Configuration and limits
create free, PAYG, PUF offers
10 active offers per sku
150,000 redemptions per app, per quarter
redemption supported on iOS 14.2 and later

## redemption
In-app redemption.  Adding the redeem code button to your app's paywall.  System automatically provides code redemption screens.  

via URL, can just specify the URL or QR code.  Deep links users directly to appstore redemption flow.  Unlike 1-time-use codes, they're not redeemable thorugh app store redemption flow.

| custom codes                     | one time use codes              |
| -------------------------------- | ------------------------------- |
| you pick the code                | alphanumeric randomly generated |
| redeemable by multiple customers | one time use                    |
| no set expiration                | expire after up to 6 months     |
Distribute your codes across online and offline.  New, existing, lapsed customers.  Apple UI handles a seamless redemption experience.  
# suggested use cases

* alongside physical goods or services
* Marketing channels (email, billboards, etc.)
* Physical retail or events
* Third-party relationships
* customer service

## Marketing channels
ex, consider running a spring break social media campaign, included in the campaign copy.  

## Physical retail or events
Consider giving event attendees an extended trial.

## Third-party partnerships
ex through influencers for example
# setting up offer codes
Custom codes may be up to 64 characters and may not include special characters.  You can't edit the code after creation.  Can't be used for other offer campaigns or subscriptions.

Redemption limit -> total number of customers that can redeem.  Up to 25k redemptions, but can use the same code multiple times on the same offer, up to 150k redemptions per app, per quarter.

Expiration date.  Select the no expiration date option from datepicker.  
UP to 10 offers active at a time.  Can't be edited once they're live.

To create a URL, copy the example link from offer details page, add custom code to end of URL.

# Engineering offer codes

## redemptions

subscribers limited to one redemption per offer code `offerIdentifier`
Each offer code is unqieuly identified by the name in ASC
moving to a lower level of service not supported
must be SKU in same or higher level

Allow subscribers to redeem offer codes directly in app
Integrate the storekit code redemption sheet
PresentCodeRedemptionSheet

Can redeem by URL.  https://apple.apple.com/redeem?ctx=offercodes&id=xxx&code=NEWSEASON

Listen at launch: Transaction.Updates
Ensures interrupted transactions and external transactions are not missed
Critical for ask to buy, PSD2, and offer code transactions
Call finish when purchase is delivered
## offer configuration

Can subscribers also receive an introductory offer when redeeming an offer code?

Intro offer: one week free
Intro offer and offer codes
What if we don't allow?

When redeeming offer code, it goes into effect.  After the offer period, sub renews at regular price.  Now the subscriber may lapse/churn.  Now they can get their 1 week free trial.  So this determines at what point they receive the introductory offer.

What if choose to allow redeeming both?
Offer code: intro offer goes into effect first, followed by offer code, then sub renews at regular price.  
## storekit

`offerType` = 3
Offer reference names: `offerIdentifier`

server apis:
verifyReceipt
app store server api
serve rnotifications 2: OFFER_RECEIVED

# Wrap up

Use custom codes for mass distribution
Optimize app experience by testing interrupted transactions in Sandbox
Support offer code redemptions within your app

Clearly communicate offer details, eligibility, expiration, and redemption steps in your communications

developer.apple.com/app-store/subscriptions/#offer-codes


