Discover the latest updates to the App Store Server API and App Store Server Notifications. Explore the current API offerings and learn how to track subscription status with notifications, work with transactions on your server, and efficiently recover missed notifications. We'll also show you how your server can support apps using StoreKit or StoreKit 2, and share an important deprecation in the API and suggested migration workflow.

* on-demand data
* get transaction history

server notifications v2
comprehensive information
* new subscriptions
* renewals, expirations, refunds, etc

Full in-app purchase lifecycle

common data format
* json
* signed by apple
* broad client compatibility
	* storekit2
	* original storekit

actively support these apis for latest features
crafted yb your feedback

see docs for more info

# transaction flexibility
transactions represent purchases on a device
purchase data
product identifier, type, purchase date, much more

endpoing that returns full transaction history.  From past to present.
sometimes your server is already aware of a transaction, ex due to a call from app to server.  Server-side, may want further validation.  Ensure we have uptodate information.

Previously you had to call getTransactionHistory.  Once found, you could refresh your record of the transaction.  This process might feel tedious, particularly if your users' transaction history spans multiple pages.  Also doesn't work if you're looking for a finished consumeable transactions as those don't appear.

Today we're introducing 'get transaction info'.
* transaction info for as ingle transactionId
* Quick and precise lookup
* Any product type, consumable/noncomsumable
* and any finished status.

Some enpdoints require originalTransactionId.

transaction hstory, subscription states, refund history, etc.  This indicates which user you're requesting or sending data for.

Might not always have original tid handy.  What if all you have is a transactionId?  You can send to new transactionInfo endpoint, to get original id.  But why call one endpoing just to call another?

Starting today, you can call these endpoints with any transactionId.  Just provide that in the path of your request, doesn't have to be original.


# Notification status and results
If your app offers autorenewable subscriptions, important to track subscription status.

5 possible statuses.

many notification events directly indicate status through type/subtype.

For some notifications, subscription status may not be so clear.  ex refund notification.  

Can be tempting to assume status is revoked.  But it might not be the case.  

Valid indicates subscriptions tatus
* active
* expired
* billing retry
* billing grace period
* revoked

Parity with get all subscription status.

New status field makes server notiications more useful than ever.  But if your server experiences an outage, app store server may not be able to reach it.

Get notification history endpoint.  
* history of app's v2 notifications
* 6 months of latest history available
* call this during server outage.

but your server may not get all notifcations outside of an outage due to network failure etc.  Optional request field: `onlyFailures`.  
Fetch only notifications that failed to reach your server
Notifications in retry process also available
Recover from outages and missed notifications

# API migration

If your app has offeredn IAPs for some time, you're likely familiar with `veryfiReceipt`.  In 2021, we released the app store server API as the new way to get IAP data from app store server.

Let's compare.

| verifyReceipt | app store server api |
| ---- | ---- |
| verify and decode receipts | Get transaction history<br>Get all subscription statuses<br>Get transaction info (new)<br>refund history<br>order ID<br>etc |
 
notifications v1.  Still supported.  But in 2021 we introduced notifications v2. 

| app store v1 | app store v2 |
| ---- | ---- |
| real-time iap events type | real-type events type/subtype |
|  | additional events |
|  | test notification |
|  | notification history |
|  | subscription status (new) |
by adopting notificationsv2, you'll unlock a wide arrya of new features for securely/efficiently managing IAP data.  Ultimately that means a better IAP experience for your customers.

* verifyreceipt is now deprecated
* app store server notifications v1 is now deprecated
* plan your migration now

Sign JWT for API authorization
see documentation for instructions
header for every API request

Save a transactionId for each user
path parameter for core endpoints

Begin calling app store server API

Migration from v1 to v2 is simple
* prepare to parse v2 notifications
* JWS format matches app store server api
Test with sandbox first
V@ notifications begin immediately
v1 retries may continue for up to 3 days

* APIs testable in sandbox
* App Store Server Library
	* Call the app store server API
	* Verify signed transaction and notification data
	* Extract a `transactionId` from a receipt
* [[Meet the App Store Server Library]]
[[Explore in-app purchase integration and migration]]

# Wrap up
* see docs
* all features available now
* share feedback with FA





# Resources
* https://developer.apple.com/documentation/appstoreserverapi/app_store_server_api_changelog
* https://developer.apple.com/documentation/appstoreservernotifications/app_store_server_notifications_changelog
* https://developer.apple.com/documentation/appstoreserverapi/generating_tokens_for_api_requests
* https://developer.apple.com/documentation/appstoreserverapi/get_transaction_info
* https://developer.apple.com/documentation/appstoreserverapi/onlyfailures
* https://developer.apple.com/documentation/appstoreservernotifications/status
