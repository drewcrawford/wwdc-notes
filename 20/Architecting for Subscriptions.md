* business
* engineering
* server-side
* data analysis

# Understanding the subscriber journey
1.  `INITIAL_BUY`, may 1.  At this point, we will continue to renew on the 1st.
2.  `DID_CHANGE_RENEWAL_STATUS`, cancelled.  Apple sends a s2s notification.  If no subsequent actions are taken, this user will voluntarily churn.  `Expires_date`.
---
1.  Alternatively, `DID_FAIL_TO_RENEW` -> Apple was unsuccessful in charging the card.
2.  `DID_RENEW` -> User updated their payment info.

---
grace period example
1.  `INITIAL_BUY`
2.  `DID_FAIL_TO_RENEW`.  However, any recoveries that happen within a 16-day period will keep the billing-day continuity.
3.  `DID_RENEW`

**Each subscriber journey is unique**.

## considerations
* upgrade, downgrade, or cross-grade events
* subscription offer opportunities
* billing retry and grace period messaging
* refunds

[[What's new with In-App Purchase]]
[[In-App Purchases and Using Server-to-Server Notifications - 19]]
[[Subscription Offers Best Practices - 19]]
[[Engineering Subscriptions - 18]]

## subscriber state
* each renewal event is a static entry
* Combinations of appReceipt JSON fields create states
* Subscription states allowed to a tailored experience

*many* different states which a subscriber can land in.  These are a combination of different receipt values.  

Also defined relative substate.

* active (auto-renew on)
* active (auto-renew off)
* non-renewing subscription
* off-platform
* expired (in grace period)
* no purchase found
* expired (in billing retry)
* expired from billing
* failed to accept price increase
* product not available
* expired voluntarily
* upgraded
* refund from issue
* other refund

### substate
* standard subscription
* free trial
* introductory offer
* subscription offer

## Retention
Need to look at **most recent receipt data**.  We may only see `active`.  
However, when we look at additional properties, we may see autorenew-off or a subscription offer.  May want to attempt to retain them.

## billing retry
* expired
* but is in billing retry -> we (apple) are trying to collect payment
* standard subscription

maybe you want to serve them a banner that deeplinks them to update payment.

## grace period
* expired
* expired in grace period
* subscription offer

maybe offer them a countdown of days remaining?  deeplink to payment info?

## winback
* expired
* expired from billing
* standard subscription

maybe send them a PN.  Deeplink the user to an appropriate payment screen?

## upgrade
Maybe upgrade the user to an annual plan instead.

This is just a small subset of all the states a user could be in and all the things we might want to do.


# Define subscription entitlement
Access.  Might vary based on geographic availability, billing state, etc.
Products.  Need to determine if user qualifies.
Offers.  Eligibility for discounted pricing, etc.
Messaging.  More meaningful than simply communicating an expiration date.

Need to build server-side logic to digest this data.  We'll call this the Entitlement Engine.

* digest data
* calculate entitlement
* Listens/responds to changes

# Crafting the Entitlement Engine
We will be providing sample code in node.js in an article with this section to represent the parts in this process.

https://developer.apple.com/documentation/storekit/in-app_purchase/subscriptions_and_offers/determining_service_entitlement_on_the_server

## prepare data
App receipt is our source of truth.
* validate receipt
* fetching the newest receipt from verifyReceipt.
* Add additional insights.

## synthesize
* Digest transaction data
* Build response
* Attach key details

## response object
* product ID
* entitlement code
* number renewals
* start date
* expiration date
* trial/offer consumption
* group id
* hours watched (outside receipt)
* upcoming evnet (outside receipt)

## cohort
* Support cohorts
* Locate subscriber

Just assign each state (from the state chart above) to some value.  Unique int.
Substates we just call a "variant".  Give those a decimal value.

Maybe use positive values to unlock access, and negative values for no access?

Entitlement code
* major- subscriber state
* minor - product variant

## use cohort and insights
apply entitlement
apply business actions
Populate response object with values

# respond
sync database
securely respond


# Using entitlement engine

## essentials
* highly available
* accurate
* failsafe
* flexible

## architecture

1.  app store server
2.  our backend
3.  subscriber (ios client)

Note we can add storage.  Now we can cache entitlement stuff for web, etc.
also add an endpoint for server notifications

# wrap up
Build a responsive entitlement architecture
Tailor subscriber experience
Enhance and repeat


