#tvOS

In tvOS 13,
* switch users in control center
* link app profiles with device users

[[Mastering the living room with tvOS - 19]]

In tvOS 14, we're letting users access their own data in iCloud.

* Participating apps have access to resources for the selected user
* When the selected user changes, your app will be terminated
* Foreground app automatically relaunched for the new selected user


Pretty much all tvOS apis will compartmentalize data by user.

# User management capability
* Run as current user
* Tells the system your app supports multiple users

# Application lifecycle
Participating apps are terminated when user changes
Save any unsafed data before the process exits
`UIApplicationDelegate.applicationWillTerminate(_:)`
System grants limited time to save data

# #CloudKit notifications
Your app may get CK notifiations for other users on the device
`CKNotification.subscriptionOwnerUserRecordID` and compare with record ID of the current CK user.
Discard notifications meant for other users.

