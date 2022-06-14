#storekit 

Last year, we introduced StoreKit2.  Modern language features including concurrency.  On server-side, we added server API.  Also released v2 of appstoer notifications.

Today, I'll be going over new APIs as well as enhancements to storekit model.  Then, exciting server updates.

# App transaction
Validate the app purchase to prevent fraud.
* signed information for a purchase of your app
* Signed using JWS
* Replaces original receipt
* StoreKit performs verification
* You *can* perform your own validation.
	* Well-documented standard.  See docs.

Updates automatically.
Provide UI to refresh.
Only refresh in repsonse to user input.

App transaction use cases
* Business model change
* Check pre-order date
* Check purchase date

Original app receipt
* receipt fields, in-app purchases, etc.

Now broken into 2 copmonents
* transaction history
	* can calculate somet hings locally?
* app transaction

One-time purchase, subscription.

Business model change: paid to free + IAP.

Here's our timeline.
1.  Initial release: paid app that cost $4.99.
2. Now free, but a variety of IAP that unlock new features.  

Types of customers.
1.  Customer who bought the paid app originally.
2. Customer who bough the free version.  

`AppTransaction.shared`.  Type contains JWS payload.
switch result.  `.unverified` => prompt the user that their app purchase cannot be verified by the app store.
"Offer a minimal experience for my app"

`verified` =>check if the user has paid for the app.
`originalAppVersion` => app version in which the user downloaded the app originally.
here we can do different properties based on when they purchased.

# New properties
* Price locale
* Server environment
* Recent subscription start date
* Sentinel values

You can use these on last year's releases!   Even though they didn't ship with them.
Available in Xcode 14 and above.  Possible because the implementation for these properties is compiled into your app, not the OS.

Keep in mind, they will return sentinel values when you're using storekit testing in xcode in older OS.  These are placeholder values.

Production and sandbox environments extract things from server response.  "StoreKit testing in Xcode" however is a local testing environment independent from the appstore server.  So we're not able to backport this to previous OS there.  So update your test device to new OS.

## Price locale
Use `displayPrice` for purchase price.
Use `priceLocale` for numbers derived from decimal price.  e.g. calculate monthly for a yearly subscription, or calculate % savings.

## Server environment
Transaction and renewal info
Xcode, sandbox, or production.
When your app generates transaction, it can be from any of these environments.  Id ont' want to add too much noise to my analytics, so this can help you filter out information.

## Recent subscription start date
* Most recent period of continuous subscription
* No more than a 60 day gap between any 2 subscribed periods
* **Not an indicator of continuous days of service**
* For loyal customers, maybe offer them a reward.  
* Offer them an incentive to use the product again.

## Sentinel values.
Placeholder values that serve as an indicator of real values.

| property                       | sentinel value               |
| ------------------------------ | ---------------------------- |
| price locale                   | `Locale(identifier: "xx_XX") |
| environment                    | String.empty                 |
| recent subscription start date | Date.distantPast                             |

only encountered in "testing in xcode" AND old OS.

recap
Available via app store server API and server notifications.
# SwiftUI APIs
## Offer code redemption.

Transaction appears in transaction listener
Set up on app launch
Available starting iOS 16

## request review
Customers might use reviews to decide whether to download the app.  Feedback, suggestions, etc.

make it easy to get info from customers.  You're listening, engaging with them, etc.

`requestReview()` API.  Get an instance of the action.  Simply call the function to display the review prompt (*why doesn't the other API work imperatively like this?*)

Prompt will only be displayed 3 times within a 365 day period
ask sparingly
Don't interrupt
Can be disabled



# StoreKit messages
Represents a sheet over your app to display message
From app store
Retrieved when app foregrounds

ex, price increase consent.   Asks the user to take some action.

1.  Your app enters foreground
2. pending messages to display?
3. SK asks appstore
4. returns info to SK
5. SK delivers to a message listener.  If your app has set it up, it sends the information to the listener.  Decide whether or not to present.
6. if you don't set up the message listener, SK displays the message right away over your app.

## ex
If a message gets delivered, it woudl be confusing to the user to be interrupted by the message sheet.  Impelment the message API to ensure this doesn't happen.

Penidng messages are sent each time your app comes to the foreground.  Set up a message listener to defer the presentation of a message.

`StoreKit.Message.messages` async sequence.  Your app could receive the same message more than once.

Environment variable `.displayStoreKitMessage`.

Then I pass to an imperative function again.

Messages may be delivered more than once.  It doesn't get marked as read until it's presented to the user.  SK deduplicates them on its side, so you can't present one more than once.

recap
* sent when your app foregrounds
* set up message listener to defer presentation
* Tailor logic based on message reason

# Application username
If you have a user account in your server, chances are you're already making use of the application username property.  This associates the transaction with account on your service.
`applicationUsername` in original API.  
We recommend that you provide a UUID.  When you provide a UUID string, SK persists the value and you will see it in updates.
If you don't provide it in a UUID, SK may not persist it.  No guarantee it will persist when the queue updates the transaction.
When you provide the string UUID, you can identify which user accuont began and completed the transaction

we implement this with `appAccountToken` which requires a UUID format.
You'll see a UUID appear in our modern APIs.  Compatible with `.appAccountToken`.  When you update your codebase, the UUID you use is preserved as an a`appAccountToken` 

# Wrap up
* New APIs
* New properties
* Use UUID for `applicationUsername`
[[What's new in StoreKit testing]]
[[Meet StoreKit 2]]

# Server
Recent developments from the past year.  Exciting new updates from the app store server API.

Last year,
* App Store Server API
* App Store Server notifications v2
* Get transaction history endpoint
* Get all subscription statuses endpoint
* These conveniently key off of the original transaction ID.
* simplify stuff on your server.
* v2 notifications, call your server directly and provde updates as they happen.
* Track changes to in-app subscriptions, etc.
* Signed JSON payloads
* Update server code to leverage future enhancements

[[Manage in-app purchases on your server]]
[[Meet StoreKit 2]]
[[Support customers and handle refunds]]

New:
## New transaction fields
* `environment` in transaction and renewal info
	* Use the environment to tell if it's in the production or sandbox environment
* `recentSubscriptionStartDate` in renewal info
	* Ignoring any gaps of 60 days or fewer
	* Understand customer's loyalty at a glance
	* For more detail, call transaction history
	* For even more detail, notifications v2, app stoer will send updates about user subscriptions.
		* Maximum insight into timing of events like pref changes, billing failures, etc.

## App Store Server API

## Get transaction history
* full history for a user
* paginated using a revision value
* sorted by ascending modified date

1.  provide `originalTransactionID`
2. Returns up to 20 signed transactions for that user.  Updated `revision` for your next page request.  `hasMore` indicates there is more data available.
3. Request again `originalTransactionID`, `revision`
4. again get transactions, NEW revision, `hasMore=false`.  
5. Note that in 4 you might receive transactions "again", this means they are modified and put at the top of the sort order.
6. What has changed?  In this instance, you notice `revocationDate` is now populated.
Good idea to store the `revision` value alongside the `originalTransactioNID`.  The next time you call the endpoint, provide that revision and get *fresh* transaction that have been modified since your request.

This API too complicated?  Returns too much data?  Even with pages.

### Sort/filter options
* Sort by descending modified date
* New filters
	* In-app product type
	* product ID
	* subscription group ID
	* purchase date range
	* many more

```
GET /inApps/v1/history/{originalTransactionID}

?productType=NON_CONSUMABLE
?excludeRevoked=true
etc
```

For follow-up requests, make sure to include the exact same query parameters in addition to the revision from the response.

Can provide multiple value to some parameters, you can define them multiple times in your URL.

## App Store Server Notifications
Track the lifecycle of autorenewable subscriptions in your app.  Retain customers, etc.

Sandbox testing environment is the best place to start.  Set seperate server URL for sandbox.

1.  Create sandbox account
2. First time buy
3. INITIAL_BUY

but what if it doesn't come?

## Request a test notification
Ask us to send a v2 notification of type `TEST`.  New test notification is used exclusively for this endpoint.  Call in sandbox or production.  Quickly test new URLs and configs.

POST request
200 response: test notification token.
your server receives TEST hook.

Contains all the usual top-level fields of a v2 notification.  Contents are shorter than a normal notification, there are no transaction-related data to include.  We omit transaction-specific fields.

Keep in mind that notifications are sent asyncronously.  Actual notification will arrive separately a short while later.

What if the test notification doesn't arrive?  To further enhance capabilities, we have an endpoint to check the status.

Troubleshoot why your server was unreachable

If send was unsuccessful, you'lls ee several different error values.  TIMED_OUT, NO_RESPONSE, etc.

These notification endpoints are simple to use and can save you a lot of trouble setting up and reconfiguring v2 app store notifications.

Servers aren't perfect, outages happen.  How do you recover when you go down?  A retry system.

1.  App store retries sending the same notification up to 5 times with a longer wait between each attempt.  These take place **only in the production environment.**

If your server is down long enough to miss the final retry.  Or, your server might be down quickly and miss just one notification.  So some of your records are out of date for at least an hour, but you don't know which ones.  We want to make it as easy as possible.

## Get notification history
* History of app's V2 notifications
* Rolling 6 months of latest history
* Filter by notification type/subtype or user
* Retry system is still available

```
POST /inApps/v1/notifications/history
{startDate:
endDate:}
```

keep in mind, the earliest notifications are 6 months before the date of your rqeuest.
Filter by subtype.  Keep in mind that some notifications have no subtype.
Filter by user.
Provide a pagination token for every followup request to get the next page.  Ensure you use the same request body for followup requests.

Up to 20 notifications, with the oldest notifications first.  Each entry in the array represents a notification.  Inside, there's a signed payload which you can decode as desired.  Identical to payload the app store server sent in the original notification.

`firstSendAttemptResult` => Look for sequences of timeouts and other errors to better understand why your server missed notifications in the past.
`paginationToken` => provide in next request to get the next page
`hasMore`

# Wrap up
* Every feature available now
* Start testing in sandbox
* Deploy to production when ready

[[Explore in-app purchase integration and migration]]


