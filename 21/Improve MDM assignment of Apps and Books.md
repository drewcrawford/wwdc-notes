# Real-time notifications
State changes to assignments, assets, and registered users
Must opt-in
Subscribe per notification type in `/mdm/v2/client/config`
Remove the need for continually syncing state

Asset is allocated to either user or device managed by an organization.

* subscribe to `ASSET_MANAGEMENT`
* Notifications triggered by associate, diassociate, and revoke asset event
* Allows MDM to provide users and devices faster access to content

## Assets
Apps and books that you've purchased.

Org cannot manage/assign assets until MDM knows they own the content.  With asset notifications you're notified about purchases, transfers, and refunds.

* subscribe to `ASSET_COUNT`
* Notifications triggere dby buy, transfer, and refund event
* Allows MDMs to display recently acquired assets promptly

get `/mdm/v2/assets?adamId=...&pricingParam`
We require `Authorization: Bearer`

## Registered users
When you need to assign content to a user (vs device) creating registered users is the first step.  Associate to personal or managed apple ID.

For personal, user must accept invitation.  Now you get notified.

* Subscribe to `USER_MANAGEMENT` and `USER_ASSOCIATED`
* Triggered by create, associate, update, and retire users event
* Allows MDMs to easily track when users have accepted notifications

`GET /mdm/v2/users?clientUserId=client-100`

* Requires MDM opt-in for each token
* Provide `notificationAuthToken` and `notificationUrl`
* Return an HTTP 2xx response to indicate successful delivery
	* Apple will retry otherwise
* Perform narrow sync to reconcile missed notifications


# Asynchronous processing
* Server enforced parallelism
* Ordered processing
* Stress-free deployments

Previously, 
1.  Your server talks to apple to manage licenses
2.  Apple returns response synchronously, with license info

This is 1 app/request, up to 10 devices/request, meaning 10 assignments/requests.

To do 250k assignments, we need 25k requests.  With new API, this becomes 10 requests.

How does it work?

1.  Request to apple.  
	1.  Provide multiple assets, up to 25 currently
	2.  Provide up to 1k devices
2.  Apple returns response synchronously.  Here we have statuscode (please interrogate).  then eventId
3.  As assignments complete, apple will post notifications
	1.  In each notifications we have some subset of assignments and metadata for the assignments.  

# Wrap up
* Real-time notifications to keep state up to date
* Asynchronous processing results in faster management
* Legacy API will still be supported
* New API is available starting today

https://developer.apple.com/documentation/devicemanagement/app_and_book_management
https://support.apple.com/guide/apple-school-manager/
https://support.apple.com/guide/apple-business-manager/

