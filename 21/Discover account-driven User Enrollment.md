Designed for BYOD.  User enrollment has a limited set of payloads/restrictions.

Available on iOS, iPadOS, and macOS.

**balances security and privacy**.

* Managed Apple ID
* Data separation
* Management capabilities

# Managed Apple IDs
* Provides access to apple services
* Owned by org ABM/ASM
* Support federation

# Data separation
managed APFS volume created during user enrollment
Unenrollment erases the volume and its cryptographic keys
# Management capabilities

* Privacy protecting
* User maintains control of personal data
* Not supervised

Full control of managed content, but can't restrict personal settings.

# Managed apple id
We improved the experience of accessing managed accounts in settings.  When a device is user-enrolled it will show the account in toplevel of settings.

Users are able to see which parts of the system are managed by org and which are not.

Managed apple IDs now support iCloud drive.  Important feature now available.  On iOS/iPadOS, shows up as a new location in files app.  On macos, additional location in finder.  

# Managed apps
* Install Mac apps as managed
* Deploy custom configurations
* Removed by comand or un-enrollment

Now for user enrollemnt.
* App data container on separate volume
* Removed during unenrollment or MDM
* Managed CloudKid apps will use apple ID associated with the user enrollment
* Set `InstallAsManaged` to true

Remember
* Need to be installed to Applications folder
* Contain a single app
* Use data protection keychain and sandboxing to ensure data is separated.

* Managed openin can now include copy/paste.
* Specify that a required app is installed without user confirmation
	* [[21/What's new in managing Apple devices]]
# Onboarding
User enrollment introduced in iOS 13 is driven by MDM enrollment profile.  has to be created for each user.

In iOS 15, we've created a new user enrollment flow that establishes user's organizaiton identity as the entrypoint.

Now, MDM can verify user before the profile is downloaded to device and any organization data is sent.  

1.  Service discovery
2.  User authentication
3.  Session token
4.  Enrollment

User is prompted to enter organization identifier.  This identifier has 2 main pieces delineated by the @ sign.  User ID and organization domain.

Device takes domain and does `/.well-known/com.apple.remotemanagement`.
Here's where you host your MDM server document.  
Device performs a GET request to that URL, expecting to get back a JSON document.  That json object includes a version key and base URL key.

Device posts to server endpoint.  Authentication response.   401 Unauthorized to challenge.

`WWW-Authenticate Bearer method="apple-as-web" url="https://mdmserver.example.com/authenticate"`

Method: what type of authentication is being used.  Today, this is fixed.
URL: TLS, endpoint on MDM server.

This kicks off the AS web signin flow.

Multiple RT between device and authentication service can occur. 

This time, the request includes an auth header which contains session token as value.  Server validates the token and returns enrollment profile.

1.  `AssignedManagedAppleID` => user required to auth with this apple ID during enrollment
2.  EnrollmentMode: BYOD.  Device verifies that this value matches the mode and will fail the enrollment otherwise


# Ongoing authentication
With new user enrollment, introducing the ability to reauthenticate at any point in time.  Makes it possible for server and client connection to be more secure than ever.  

MDM server can validate authentication at any time.  via SessionToken.

## Persistent authorization.
Session token is sent in every request

## Refresh session token
Server can request user reauthentication at any time

By implementing these, you can add your organization security rules into management solution.  e.g. periodically verify credentials to ensure sensitive payloads are only sent to devices with trusted users.

1.  Server invalidates token locally
2.  Next device checkin triggers 401 response
3.  Response that the server sends includes `WWW-Authenticate: Bearer ` etc.
4.  Reauthentication is surfaced to user via notification center
5.  User interaction
6.  Client will start another signin flow like during onboarding
7.  Server sends redirect response with custom location header and new session token
8.  Successful reauth repeats checkin request, continuing where it left off.

If, for any reason, a user's auth request fails, the server may no longer trust the device.  Server can remove any sensitive MDM payloads or unenroll the device.

* HTTP 401 does not trigger unenroll
	* Only triggers reauthentication
	* This is for user enrollment only
* `RemoveProfile` to unenroll
* 401 response for all profile based enrollment is unchanged
# Get started
1.  Set up a discoverable web domain
2.  Identity provider integration
3.  Managed apple ID
4.  Update MDM payloadDevice management documentation

# Wrap up
* use iCloud Drive
* Leverage managed apps
* Support for account-driven onboarding
* Incorporate ongoing authentication

