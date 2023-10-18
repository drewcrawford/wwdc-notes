Engineer on apple push notifications team.  Push notifications console.  Combines a few features that will help you integrate push notifications into your app.

* provide updates
* dynamic experiences
* Increase engagement

apple push notification service - APNs
prompt the user to allow notification sfrom your app.  If the user agrees, a device token ins generated and then sent to the device.

Keep tokens up to date on the server-side.  

console is a brand new tool.  If you're developing an app, and adding push notifications to it, you might want to test end-to-end.  That's where the feature comes in handy.


# Send notifications
# Examine delivery logs
in-app notifications are not erceived, leaving you uncertain about what happened to it.  Help you analyze these cases.

events that reflect delivery process are recorded.  

Enable low-power mode, to simulate reason where notifications is not received.

same information is available on the send page itself.
# Debug with tools

two different authentication strategies
* certificate-based
	* SSL Certificates to establish a trusted connection.  Create/manage certificates for each app/environment
	* certificates expire and need to be renewed periodically
* token-based
	* JWT
	* generate token with private key associated with developer account
	* do not expire
	* now a tool that can generate authentication tokens.  supply a private key obtained from the developer portal, and associated key ID.
	* validity period of these tokens cannot exceed 1 hour
	* key not uploaded anywhere
	* tokens can also be validated.

# Wrap up
* send push notifications
* analyze push notification delivery logs
* validate and generate tokens

[[The Push Notifications Primer]]



# Resources
* https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/establishing_a_certificate-based_connection_to_apns
* https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/establishing_a_token-based_connection_to_apns
* https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns
