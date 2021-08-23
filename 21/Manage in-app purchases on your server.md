# Why have servers?
* Receive status change notifications
* Track status changes through APIs
* Validate access anytime
* Track refunds for IAP

Server-to-server communication.  

# Validate status with receipts
* On-device receipt validation
* Server-to-server `/verifyReceipt`.

Can be huge.  Contains transactions from entire app.  Ton of info, but maybe it's too much?

SK2 introduces new JWS format, we provide the same on the server.

## JWS
* Enhanced security
* Easy to decode
* Verify the signature on your own

```
Base64(header) + "." + Base64(payload) + "." + sign(Base64(header) + "." + Base64(payload))
```

One date format
Made more things numbers
appAccountToken => app customer ID
cancellationDate/reason => revocationDate/reason
offerType/offerIdentifier simplified version of many prior fields.  1=intro,2=promotional,3=offer code

## Verifying
Use the header portion of the JWS transaction ifo
Determine signing algorithm using `alg`
Use certificate chain in `x5c` claim
Use your favorite library to verify

# Check status with APIs
We wanted to build APIs to help even if they're not strictly necessary.

Two new APIs:
* Subscriptions tatus API
* IAP history API

## Status api
Latest status for subscriptions
Request status for a specific `originalTransactionId`

status
* 1 - active
* 2 expired
* 3 billing retry
* 4 - grace period
* 5 - revoked

Can verify signature of signed renewal info in the same manner using the JWS header etc.

## IAP history API
Provies history of all in-app purchases
Replacement for `latest_receipt_info` in `/verifyReceipt`
API pagination to control response size

How all server APIS can be consistent?
* JWT authentication
* JWS transactions
* JSON requests and responses
* based on `originalTransactionId`

## JWT authentication
Standard authentication for all app store server APIs
Generate private keys in ASC
Sign using ES256

1.  Header
	1.  key id
	2.  algorithm: `ES256`
	3.  type: `JWT`
2.  payload
	1.  issuer ID: in ASC
	2.  iat => issue time
	3.  exp => expire time
	4.  audience
	5.  `appstoreconnect-v1`
	6.  nonce
	7.  `bid`: bundle identifier
3.  signature

* Separate status and history functions
* Require only `originalTransactionId`
* Take the signed transaction and store the individual fields
* No need to store the signed transaction

# Track status with notifications
* Receive notifications when status changes
* Update status immediately
* No need to ask for status

## v2
* Only one notification per subscription event
* Update payload to be basedon signed transaction
* Sign entire payload in JWS
* Opt into v2 notifications when ready

Currently, 11 total types.  With v2, we're deprecating 4 of them.

INITIAL_BUY, INTERACTIVE_RENEWAL, CANCEL, PRICE_INCREASE_CONSENT

Adding SUBSCRIBED, OFFER_REDEEMED, EXPIRED, GRACE_PERIOD_EXPIRED, PRICE_INCREASE

## Notification substates

| Notification type           | Substate value                                                         |
|-----------------------------|------------------------------------------------------------------------|
| `SUBSCRIBED`                | `INITIAL_BUY`,`RESUBSCRIBE`                                            |
| `DID_CHANGE_RENEWAL_STATUS` | `AUTO_RENEW_ENABLED`,`AUTO_RENEW_DISABLED`                             |
| `DID_CHANGE_RENEWAL_PREF`   | `DOWNGRADE`,`UPGRADE`                                                  |
| `OFFER_REDEEMED`            | `AUTO_RENEW_ENABLED`,`INITIAL_BUY`,`RESUBSCRIBE`,`UPGRADE`,`DOWNGRADE` |
| `EXPIRED`                   | `VOLUNTARY`,`PRICE_INCREASE`,`BILLING_RETRY`                           |
| `PRICE_INCREASE`            | `PENDING`,`ACCEPTED`       

### `SUBSCRIBED`
| Subscriber event             | Notification type + substate |
|------------------------------|------------------------------|
| First time purchase          | SUBSCRIBED + INITIAL_BUY     |
| Resubscribe to same SKU      | SUBSCRIBED + RESUBSCRIBE     |
| Resubscribe to different SKU | SUBSCRIBED + RESUBSCRIBE     |

### `OFFER_REDEEMED`
| Subscriber event                             | Notification type + substate        |
|----------------------------------------------|-------------------------------------|
| offer redemption results in a first-time buy | OFFER_REDEEMED + INITIAL_BUY        |
| offer redemption to subscribe to same SKU    | OFFER_REDEEMED + RESUBSCRIBE        |
| offer redemption to upgrade subscription     | OFFER_REDEEMED + UPGRADE            |
| Offer redemption to downgrade subscription   | OFFER_REDEEMED + DOWNGRADE          |
| Offer redemption re-enables auto-renew       | OFFEr_REDEEMED + AUTO_RENEW_ENABLED |

### `EXPIRED`
| Subscriber event                                  | Nofication type + substate value |
|---------------------------------------------------|----------------------------------|
| Subscription expires after customer cancels       | EXPIRED + VOLUNTARY              |
| Subscription exits billing retry without recovery | EXPIRED + BILLING_RETRY          |
| Customer did not consent to a price increase      |  EXPIRED + PRICE_INCREASE        |

### v2
Over 20 different in-app events
Substate offers more granular detail

Same fields regardless of notification type.
1.  notificationType
2.  subtype
3.  version (2)
4.  environment
5.  bundleId
6.  appAppleId
7.  bundleVersion
8.  signedTransactionInfo
9.  signedRenewalInfo

Opt into v2 notifications in app store connect. Vary based on production vs development server.

## examples
### first time purchase
1.  Get signed transaction info
2.  Send to server if desired
3.  SUBSCRIBED+INITIAL_BUY

### renewal
1.  DID_RENEW
2.  Can call status API if desired

### grace period and billing retry
1.  DID_FAIL_TO_RENEW
2.  GRACE_PERIOD_EXPIRED
3.  EXPIRED + BILING_RETRY
4.  DID_RECOVER (if recovered, at any time)
5.  Can call subscription status or history API at any time to double-check your status

### first-time consumeable purchase
1.  Get signed transaction info
2.  Send to server if desired

Suppose customer requests a refund

1.  REFUND
2.  can call in-app history if desired

### outages

1.  Call in-app history for each customer
2.  Get the latest history of transactions
3.  Can call subscription status to get latest status

### Migrating to signed transactions

Only requires an original transaction ID.  Convert unified app receipts to JWS receipts.

1.  call verifyReceipt
2.  Call in-app purchase history for some transaction ID
3.  Call subscription status API to get subscription original status ID
4.  Record relevant data
5.  All set

# Manage family sharing
* Auto-renewable subscriptions and non-consumables
* `inAppOwnershipType`
* Notifications for family members
	* DID_CHANGE_RENEWAL_STATUS
	* DID_CHANGE_RENEWAL_PREF
	* DID_RENEW
	* INTERACTIVE_RENEWAL (maybe deprecated?)

Now adding
* SUBSCRIBED
* EXPIRED
* GRACE_PERIOD_EXPIRED
* OFFER_REDEEMED

# Test in sandbox

We want you to feel confident in your app and server.  

Fully testable in sandbox and live, **starting now**

Later this year, can add sandbox-specific URL in app store connect.  Keep production and sandbox notifications completely separate.

New features
1.  Clear purchase history
2.  Change sandbox account region
3.  Adjust subscription renewal rate
4.  Enhance `/verifyReceipt` for TestFlight
	1.  error when no longer a TF user

Most of this is done in ASC, "Testers" page.  Under "Users & Access".


# Wrap up
* Adopt new JWS transactions
* Chekc out new App Store Server APIs
* Enroll in App Store Server Notifications
* Use new sandbox testing features

[[Meet StoreKit 2]]
[[Support customers and handle refunds]]
[[What's new with In-App Purchase]]
[[In-App Purchases and Using Server-to-Server Notifications - 19]]

Powerful tools to manage IAP on your server.  Make your server and app more powerful.

* https://developer.apple.com/documentation/appstoreserverapi
* https://developer.apple.com/storekit/
* https://developer.apple.com/design/human-interface-guidelines/in-app-purchase/overview/introduction/
* https://developer.apple.com/in-app-purchase/
* https://developer.apple.com/documentation/appstoreservernotifications
* https://developer.apple.com/app-store/subscriptions/
* https://developer.apple.com/documentation/storekit




