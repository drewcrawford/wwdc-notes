Explore the latest updates to Managed Apple IDs and learn how you can use them in your organization. Take advantage of additional apps and services available to Managed Apple IDs, discover the Account-Driven Device Enrollment flow, and find out how to use access management controls to limit the devices and Apple services that Managed Apple IDs can access. We'll also show you how to federate with your identity provider to automate creation and sync with your directory.

Review.
* apple ID for businesses or schools
* sign into devices, apps, services
* owned by an organization
* create/manage with ABM/ASM.

# Features and enrollment

[[Deploy passkeys at work]]

Whiel MIDs already support messages, stocks, news, siri, now can keep data synced across all devices.  Introducing wallet this year.  

And continuity.  Sidecar, etc.  Just sign in.

What about BYOD organizations?  Now MIDs allow you to access work data on personal devices using account-driven user enrollment

[[Discover account-driven User Enrollment]]

How it works?  Personal/work data cryptographically separated by partition.  Managed appleid can use iCloud.  General, VPN&Device Management.  "sign into wor or school account".  Use your MID to sign into the organization.

But we want to offer something more for orgs that own devices.  They usually use device enrollment.  We are also offering account-driven *device* enrollment.

Users don't need to download/install profile for device enollment.  For devices through either flow, we ahve updates for 'signi nwith apple at work and school'.

Use MID to sign into managed apps that use sign in with apple.  Use your work account for work apps, etc.

If app uses a webview for authentication, or safari, click 'use a different appleID' lets you use a managed apple ID to complete signin.

When enrollment occurs, mnaged appleID is passed along as query parameter.  Now we also send device model.  New information will help you decide which users/devices receive user or device enrollment.

Server responds with `mdm-byod`.  as value of `Version` key.  In enrollment profile, the value of the EnrollmentMode is set to byod.  To use account-driven device enrollment, simply change 'version' key to `mdm-adde`.  And in the enrollment profile, set enrollment mode to `ADDE`.

That's it.

* enroll with managed appleID
* Management capabilities of device enrollment
* personal and work data separated on device
* Available on iOS, iPadOS, macOS
* results in device being supervised

# Access Management
Set access policies for managed apple IDs
Control sign-in based on level of management
Disable access to iCloud
Configure in ABM/ASM

'Any device' -> no management
'managed devices only'
'supervised devices only'

allow messages, calls from anyone or 'organization only'.  Or disable.

AppleSeed for IT
Control access through xcode and apple developer

Disable iCloud for any supported apps/services for managed apple ID.  When you disable iCloud, policy set by organization is reflected on device.

Developers must implement check-in request message type that returns a secure token.  Token will verify that you only sign into MDM managed devices.

GetToken Check-in request.
MDM need server UUID for the server.  Available in 'get account detail' endpoint.  Also need private key for MDM server certificate.

Admins set the access management policy in ASM/ABM.  Applied to all ids in the organization.

Device requests token using GetToken.  

Server will respond with JWT signed by the private key.  Device uses this token to check the policy.  

User successfully signed in.

* iOS 17, iPadOS 17, macOS 14
* Automatic signout when device no longer complies with policy
* Relationship between organization and device is proven by GetToken message

`com.apple.maid`.  Response is JWT.  

iss -> MDM server UUID
iat -> timestamp from server
jti -> sever-generated identifier, random string
service_type: `com.apple.maid`.



# Identity providers
Creating and integrating them with identity provider.

can also use identity provider to help you create them.  When you have an identity provider you can use your own domain
Enable federated authentication
Sync user directories
Microsoft Azure Active Directory
Google Workspace

**Custom Identity Providers**.  Any public or in-house provider.  

OpenID connect for federated auth
SCIM for directory sync
OpenID shared signals framework for account security events

okta, later this year.  

# Wrap up

* explore new iCloud and continuity features
* Enroll with account-driven device enrollment
* Manage access to devices and services
* Integrate identity providers
* 


# Resources
* https://support.apple.com/guide/apple-business-manager/
* https://support.apple.com/guide/apple-school-manager/
* https://support.apple.com/guide/deployment/welcome/
