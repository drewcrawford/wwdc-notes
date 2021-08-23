# Device authentication
Identity often starts on the login screen.  
Last year, we added enrollment customization, which allows a webpage to load during setup.  User info from the authentication process can then be used to populate local accounts.

Local accounts
Network/mobile accounts

## Local accounts
* Recommended whener possible for 1:1 deployments
* User record stored locally in macOS
* Search of truth for authentication is local directory services

Consider not binding to directory service
Enabling directory connectivity with built-in kerberos, SSO, or other vendor can provide "directory connectivity"

Access domain resources and sync password
Binding required for traversing DFS and AD certificate payload
Even if bound, consider a Local Account.

Offer best experience, use whenever possible

## Network accounts
Source of truth for authentication is a directory service
Home folders optionally on a network volume
Modern apps, file sizes/types, and mobility present challenges for network accounts
"We view these as legacy"
Not recommended
Not deprecated either
Use local accounts whenever possible.

## Mobile accounts
A type of network account
More common than true network accounts
Source of truth for authentication is a directory server, but credentials cached locally
Home folder is local
Directory availability can cause login delays
Additional compelxities exist in synchronizing passwords
Supported, but not recommended for 1:1
Appropriate for labs and can auto-expire

# Single sign-on extensions (SSO)
## Recap
* Introduced last year
* Enables native apps and WebKit authentication
* Leverages existing credetials
* Requires MDM management
* UI can be native, web, or non-existent

## New
* User-channel profile delivery
* Per-app VPN Improvements
* Embedded Wildcard Matching
* Additional calling app information
* Profile removal operation
* Kerberos Extension updates

### User-channel profile delivery
* Supported on both iPadOS and macOS (not iOS?)
* Enabled for all SSO Extension
* Takes priority over system settings
* Easier to configure specific user settings (such as setting username)

### Per-app VPN
* Associated Domains work with per-app VPN
* SSO redirect extensions can redirect to on-premise IDPs, note we didn't relax TLS requirements
* New direct downloads flag

#### Example
1.  OS requests site-association file
2.  Native app can send operation to extension
3.  SSO talks to identity provider over per-app VPN

All traffic (for the app?) routed through the VPN
Per-app VPN Excluded Domains list allows for direct device-to-cloud IdP traffic.

This is useful whn an app needs access to both trusted files and on-prem resources.

#### excluded domains
Previously, SSO had to send over per-app vpn, and then routed to the internet.
Now can correct directly to trusted internet resources via excluded domains.

#### additional
Per-app vpn also triggered for `auth-only` requests.

#### associated domains
* Uses MDM to configure domains associated with SSO extensions
* Helps with different hostnames per-tenant, rather than including in app payload
* Needs entitlement `com.apple.developer.associated-domains.mdm-managed`
* Now public, no longer requires approval through DTS.

Must be paired with an `apple-app-sites-association` files on the server.
No longer accessed by the device by default
Accessed via CDN now.  
Additional options available for managed apps and devices for non-internet facing domains.
[[What's new in Universal Links]]

### Calling app info

```swift
var localizedCallerDisplayName: String
var callerTeamIdentifier: String
var isCallerManaged: Bool
```
SSO extensions can use to improve user experience, and teamIdentifier / managed to make more informed security decisions

### profile removal info

```swift
// existing operations
static let operationLogin: ASAuthorization.OpenIDOperation
static let operationRefresh: ASAuthorization.OpenIDOperation
static let operationLogout: ASAuthorization.OpenIDOperation

//new this year
static let configurationRemoved: ASAuthorizationProviderAuthorizationOperation
```
Called when profile configuring extension is removed from device.  SSO can logout, delete tokens, etc.
Requires SSO extension to be built with new SDK

### embedded wildcard matching
* Easily match multiple customers
* Eases setup process for large providers with common URL schemes
* Works together with associated domains

### Improvements to built-in kerberos extension
Now when a requirement is missing, it will indicate to the user what the problem is in "menu-extra"
Customizable UI to explain the name of the account if your corp has a special name
Help info with e.g. a custom text
Better support for Per App VPN
User-chnnel supprot enables certificate-based kerberos or PKINIT
More control over first-login experience
iOS managed app access control (e.g. can prevent unmanaged app from accessing)

# Managed Apple IDs
Apple Business Manager (ABM), Apple School Manager (ASM)
* Managed apple IDs -> apple accounts, owned and managed by an organization
* Provide access to apple services
* Required for User Enrollment, Shared iPad
* Support for education features
* Support for Federated Authentication

## Federated Authentication
* Simplified identity management
* Identity provided by Microsoft Azure Active Directory
* No additional sign-in required
* Automatic account creation
* Reduces account-related overheads for IT and end-users

1.  Set up with ABM/ASM
2.  Set up per-domain
3.  Apple will recognize the user as federated and redirect to Azure for authentication
4.  Device redirected to apple with authentication token from azure
5.  Apple verifies token

Works even if the identity is not natively on AD such as on-premise ADFS or similar.

### federated setup
Requires the user to be an admin in both ASM/ABM and AD.
Per-domain.  If you have multiple domains, decide which domains to federage (e.g. pick all of them)
Domain verification happens in ABM/ASM.
Generate a verification code that you put in DNS.  Complete within 14 days.
Trigger a DNS check manually, or just wait.

Start federation using "connect" button.  Sign into AD as an administrator.  Global Admin, Application Administrator or Cloud Application Administrator.  
Accept prompts.

Do a test signin.  Confirms you can sign in with credentials.

handle conflicts.  Apple has existing accounts with usernames in your domain.
Apple will notify users to select a new username in a different domain.  After 60 days, any remaining users will be automatically moved to a temporary domain by Apple.
Once that is done, you can create managed apple IDs with any domain you are federated.
Even if there are conflicts, you do not have to wait 60 days to enable federation.
You can enable federation any time.

### recap
1.  Verify domain
2.  Provide administrator consent
3.  Sign in as a user in the domain
4.  Review conflicts and initiate resolution
5.  Turn on federation

### Problems
* User leaves the organization
* User information gets updated
* Best way to ensure user's profile information is consistent?
* User needs service specific privileges

### "SCIM"?
Autoamted process for exchanging information between identity provider and service provider.
* Synchronize user information automatically between ABM/ASM and AD
* Standards based data exchange
* Available for all federated organizations
* Educaton organizations can choose between SIS/SFTP and SCIM.

#### process
1.  AzureID monitors userid for changes
2.  publishes changes to ABM
3.  ABM consumes this data and applies to its database
4.  When keeping repositories in sync, SCIM enhances etc.

#### enable
Enable SCIM in ABM/ASM.  This is in "Data Source"
Generates SCIM token that you have to upload to AD.
Token must be used within 4 days
In AD, can decide who gets synced.  All users, or only users with a federated appleID.
Account information will be automatically synchronized now.

# Federation vs SSO
* Complimentary technologies
* Identity providers can support SSO extensions
* Use SSO extensions for authentication to non-apple apps and websites
* Apple services rely on managed apple ID with federation to authenticate

# Summary
* Use local accounts on macOS where possible for 1:1 deployments
* Enable directory connectivity ?
* Mobile accounts are supported, but not recommended for 1:1. 
	* Often appropriate for shared lab situations
* Improvements to SSO and kerberos.  Take advantage
* Use managed apple IDs in your institution
* Verify domains in ASM/ABM and enable federation
* Use SCIM to more easily handle account creation, modifications, and deletion

[[20/What's new in managing apple devices]]
[[Deploy Apple Devices Using Zero-Touch]]
[[Introducing Extensible Enterprise SSO]]
