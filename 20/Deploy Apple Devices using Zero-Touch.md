We've removed several set up assistant screens.

Devices auto-enrolled in first boot.  So we can push apps during initial setup process.

Create local account.

users self-configure various settings.

20k devices per MDM admin!

Apple drop-shipped each WFH covid employee a new device.  Users were able to power-on and quickly become productive.

# Setup and infrastructure

* ABM
* Device management (3rd party mdm)
* core technologies

During purchase, device details are sent to ABM.  
Enrollment checks can be performed
Device purchase record transferred into ABM.
A device is associated with its intended MDM server based on ABM assignment.

MDM can apply prestage profiles to the device.
Devices may be manually assigned to an MDM service that is != default MDM target for your org.

In recent releases, APNs now supports proxy configurations.
APNS traffic now honors proxy auto-config (PAC) files.
APN traffic will now use a web proxy.
Improved support for default deny networks, typical in regulated industries.
Enables MDM on proxy networks.
Workw ith your network teams to define the PAC URL in a profile.
APNS traffic cannot be inspected.

## MDM capabilities
* various
* auto wifi
* setup email
* install apps
* filevault

Will the environment be on-prem or cloud-based?
Multiple MDM instances -> staging environment etc.
Containers or VMs.
Properly-size db transactions
Use of bare-metal servers for resource-intensive functions
Firewalls
Load balancers.  But may not be necessary for small orgs.


# Deployment
Dropshipped from the supply chain directly to users.
Streamline Setup Assistant.
Automatic MDM enrollment
Self-service app installed on device.  Commonly-used apps including SSO are automatically pushed to each device.
You should intentially decide which apps are automatically pushed and which ones the user should einstall
Provide local admin rights.  UX is enhanced when users have local admin rights.

Payloads - many

# Identity and security
Identify via SSO authentication client
LDAP integration
Role-based apps and settings
Various identity providers can be used
Custom scripts
[[Leverage Enterprise Identity and Authentication]]

MDM can remotely wipe a mac.  IT can also remote lock.
# Content distribution
* deploy apps directly to devices
* Install apps without an apple ID
* Content caching -> locally caches various apple things
* Managed apps in macOS

[[20/What's new in managing apple devices]]
[[Custom app distribution with Apple Business Manager]]

# Wrap up
* Use zeor-touch practices to make your life easier
* Ensure a great device experience for your user
* Achieve a hands-off hardware deployment
* Customize settings where appropriate
* Tailor to your organization
* Scale to your needs