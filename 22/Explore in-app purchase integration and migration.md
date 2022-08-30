how to migrate to server API
how to migrate to notifications v2

# SErver and API notifications
* powerful, secure, efficient server-to-server endpoints
	* Verify that the data you receive is untampered with, signed by app store, etc.
	* Fetch any set of transactions that you specify with just an original transaction ID.
* Notifications v2 notify of entire lifecycle in real time

[[Manage in-app purchases on your server]]
[[Support customers and handle refunds]]

# Using the App Store Server API
## how to use
* StoreKit
* StoreKit2
* both simultaneously.

### Original storekit?
First off, let's see what requests to the api look like.  `/inApps/v1/subscriptions/{originalTransactionId}`.

lookup order ID endpoint.  This uses the `orderId`.  Assist with questions from the customer, as they have this value.  Not provided in `originalTransactionId`.  Respond to customer inquiries with data they have.

notification-related, discussed later.

`original_transaction_id` field in reesopnse payloads.

1.  Obtain app receipt and put on server
2. Call `verifyReceipt`
3. Returns decoded receipt.  Gather transaction IDs, etc.
4. Call `originalTransactionId` style calls.
5. Returns a history of signed transactions
6. Can get latest signed transactions for subscriptions, etc.

Looking at server side, here's an example of signed JWS transaction.  
## StoreKit 2
First, take signed transaction on device.  With SK2 you can verify on device.  

1.  Send transaction to your server
2. Can keep up to date via notifications if desired
3. Can use `originalTransactionId` to call endpoints and get back data you need

## Both simultaneously

* Original StoreKit: find original transaction ID in receipts
* SK2: find original transac tion ID in signed transactions
* server API can be used independently of any other APIs
* We do recommend using v2 because it notifies you of changes to subscriptons as they occur, etc.
* You can use server API separately with v1 notifications, etc.

To suport new purchases, you can take new receipts as they come, send to your server, and do the same steps.  


## Signing JSON web tokens
* authenticate you when calling to the app store server API
* Included in every request as an authorization header
* Consists of a header, a payload, and the signature

Base64(header) + . + Base64(payload) + . + Sign(Base64(header) + . + Base64(payload))

`kid` => your private key id in asc.  Needs to match key you used to sign.
Payload => additional information about your app.  Refer to article.
See article for details on each field.

Once you have header/payloads, can sign.
1.  Get private key
2. 2.  Call some sign function
3. JWT library signs with the header you specified

Now you can use Bearer (token) etc.


## Verifying signed transactions
* signed transactons are in the JWS format
* Verification of authenticity

1.  Decode the header
2. Determine signing algorithm using the `alg` claim
3. Verify certificate chain in the `x5c` claim

x5c chain => chain of certificates.
Subsequent certificates are signed by previous certificates
First certificate is root certificate, Apple certificate.
Leaf certificate is leaf certificate, signs JWS.

x5c => with certificates listed in order.  Now let's look at what generating an x5c looks like.

1.  Root certificate from apple autohrity
2. signs an intermediate signing certificate
3. signs a leaf certificate

To verify, we do this in reverse order.  Leaf, intermediate, root.  Root should match the one from the apple certificate authority.

How to do this?

```
openssl verify -trusted AppleRootCA-G3.pem -untrusted AppleWWDRCAG6.pem leaf.pem
```

untrusted flag => provide certificates that you wish to verify using the certificate yout rust.  We pass the WWDR.  This shoul dmatch the second cert in the x5c chain.

leaf certificate => last certificate.  In the case of a successful verification, you get a success code.  Then you use the decoded information.

If JWS cannot be validated, do not use.


## Migrating from verifyReceipt
* check changes to a subscription
* Original StoreKit: call verifyReceipt, determine status based on fields
* StoreKit 2: get subscription status endpoint
* Informs you of what the user has purchased, renewed, etc.

Previously, you had to call verifyReceipt and use in_app array and examine latest_receipt_ifno section.  With App store server API, you just call Get Transaction HIsotyr endpoint with filters.

[[What's new with in-app purchases]]

1.  Call getTransactionHistory
2. returns signed transactions

How to adopt app account token.
* associate transactions with accounts
* Original StoreKit: Use a UUID applicationUsername
* SK2: Use a UUID appAccountToken

# Server notifications, v2
## Setting up
Messages we send whenever certain notificationsare taken.
* subscriptions
* refunds

fill in gaps into user actions that may nto be available in the app.  ex, renewal.  A user may not be in the app when this transaction is available.  Proactively send latest notification to your servers.
* compatible with original storekit
	* continue to support clients where sk2 is not available
* comprehensive in-app purchase data

This presentation doesn't tell lthe whole story [[Manage in-app purchases on your server]] [[Support customers with StoreKit 2 and App Store Server API]]

1.  ASC.  App Store Server Notifications.  Here you see production/sandbox URLs.
2. We recommend, especially if you're v1, that you try v2 in sandbox.
3. Valid HTTPS certificate
4. Allow Apple IP addresses 17.0.0.0/8
5. Use request a test notification endpoint.  Whenever you're performing a configuration change.

Just like transactions, notifications are also in JWS format.  Decode and verify
1.  Extract signed payload field
2. same steps as earlier to verify signed transaction.
3. Verify which app the notification is for.  If you have multiple apps sharing the same endpoint.  Also improtant to confirm that the app is your app, and the notification was not intended for another developer.
4. environment of the notification matches your expected environment (production or sandbox).  Possible to enforce this environment, or process sandbox/producton separately

We recommend async processing.  If you take too long, our server will record a timeout.  We will then resend the notification.  Therefore, doing intensive processing later helps ensure we delivered successfully.

Body fields.
1.  notificationType, subType.  What has changed since last notification, why these changes occurred.
2. UUID => unique identifier for notification.  If server retries, this will be the same.  Helps detect cases where you processed the notification but didn't 200 in a timely manner.
3. signedDate => when notification was created.  Useful for retries
4. appAppleId, bundleId => detect target application.  Check to confirm they match expected values.
5. environment => Ensure sandbox vs production etc.
6. signedTransactionInfo and (optional) renewalInfo.  Latest status.

## Migrating to v2
adopts a different philosophy about the state of the purchase.  Instead of sending full history, we send only the latest information.

|                                           | v1           | v2       |
| ----------------------------------------- | ------------ | -------- |
| number of transactions included           | 100 recent   | current  |
| Minimum required SK version               | original     | original |
| Unique customer events and status updates | 10 scenarios | 28 scenarios (and counting!)         |

We work to provide ifnormation on every step of the lifecycle.  Notifications only contain latest info.  Together, these notifications create a complete timeline.  If you need to view the entire transaction hisotyr and don't have access, you can call `getTransactions` to query the full history.

To illustrate the complexity of scenarios, let's look at how notifications can inform each type of subscription case.

1.  New customer
2. subscribed, initial_buy, etc. => renewing subscription
3. didrenew => 2
4. did_change_renewal_status => expiring
5. expired voluntary => expired

Full lifecycle shown here.   a lot going on.  Doesn't even tlel the whole story, refund isn't included on chart shown.  Vast array o fscenarios that we cover.  

Single source of tracking subscriptions and improves confidence you're seeing everything.

You don't need to work with every type available.  Even a few types can provide value, if you're just getting started.  
## Notification recovery
Suppose your server starts failing.

if you don't receive a status code, we will retry notifications "according to our documented retry policy".  1hr, 12hr,24,48, etc.

how to tell it's a retry?
1.  signed date.  If you see a notification with singing date earlier, you may have experienced an outage.
2. Check UUId to make suer you didn't process.  We may think it failed if we got non-200 or took too long.

Can also call `getNotificationHistory`.  This provides a 6 month ihstory of notifications.  [[What's new with in-app purchases]]

After an outage is resolved, note the start and end timestamps of the outage.  Allows queries to be made over a given timespan.  Don't have to page through full history.  Improve speed of recovery, etc.

can filter by type.  If you have experienced an iextended outage, consider filtering by type and starting with types that have immediate impact such as DID_RENEW and EXPIRED.  Take action on most relevant cases first.

If notification subtype is omittted, will only return notifications that do not *also* have a subtype.  

Can filter by a specific original transaction ID.  May want to check specific customers ina  support context etc.  Or you get updates that don't make sense, indicating you missed intermediate notifications.

### pre-existing users
* history before adoption of notifications
* gaps beyond get notification ihstory endpoint
* use get transaction history for these cases.


## Gaining insights
subtype rule.
* more granularity and additional context

ex expired.  Was it due to a billing issue?  voluntary choice?  etc.

did change renewal status.  Great opportunity to win back the customer before the subscription expires.  Provides insight into customer behavior.  Is it the day before enewal.  etc.

Flows that do not change history.  ex, deactivate and reactivate before it expired.  It occurs within a subscription period, so there's no effect on long-term status.  Can help you understand your customers etc.

Notifications work to enhance and create opportunities for understanding customer behavior.  Covering more scenarios than ever before.

# Wrap up
* Provide new capabilities
* Usable with any client framework
* Available for immediate use
* 
* https://developer.apple.com/forums/tags/wwdc2022-10040
* https://developer.apple.com/forums/create/question?&tag1=129&tag2=474030
* https://developer.apple.com/documentation/appstoreserverapi/creating_api_keys_to_use_with_the_app_store_server_api
* https://developer.apple.com/documentation/appstoreserverapi/generating_tokens_for_api_requests
* https://developer.apple.com/documentation/appstoreservernotifications/responding_to_app_store_server_notifications
* https://www.apple.com/certificateauthority
* https://developer.apple.com/documentation/appstoreserverapi

