Learn how you can help IT administrators get the tools they need to manage their organization's devices. Discover the latest changes to declarative device management, including software update management, additional asset types, status reporting for FileVault, and more.

autonomous and proactive.

MDM + declarations

**The focus of future protocol features will be declarative device management**.

DDM has generated much excitement amongst mdm developers and administrators.  Several developers have already implemented and shipped MDM servers with this support.  With the new features in this release, even more... provide customers with enhanced management featureset

[[Meet declarative device management]]
[[Adopt declarative device management]]

update on platform support

[[Meet device management for apple watch]]

collaboration between many different teams within apple, providing robust and secure solutions for mdm developers and enterprise administrators.  Our focus is on providing a seamless user experience taht protects the privacy/security of users and their devices, while giving admins... to protect enterprise data.  And to enforce compliance with policies.

Your feedback helped us prioritize in this release.

# Enforce software updates

Keep devices up to date.
Enforce software updates
Defer software updates
Verify devices are up to date

MDM commands to isntall software updates
poll for software update state
defer software updates

## Declarative software updates

* configurations
* predicates
* asynchronous status reporting

`com.apple.configuration.softwareupdate.enforcement.specific`
Status:

`softwraeupdate.install-reason`, etc.

past-due notifications, etc.

can coexist with MDM software updates
but declarations take precedence over MDM commands/profiles
available for macOS, iOS, iPadOS.

# manage apps

licsenign, installation, updates, support
removing apps, etc.

MDM has install, remove, list commands
server apis for metadata
imperative, polling app support
user-experience latency

**declarative apps**.

configurations
predicates
on-demand install
asynchronous status reporting

configuration: `com.apple.configuration.app.managed`
status: `app.managed.list`.
app store and enterprise apps
macOS packages supported
only one app per package
app installs can be optional (triggered by user)

MDM:
User control of management
Lists available apps
Tap to install on demand
Private protocol for app and server

New managed apps view
managedAppDistribution framework.
Entitlement required.  can be applied for as part of app store submission process
SwiftUI view extension for managed apps
Tap to install, update, etc.
Immediate install and progress

Sort and group
Use in your app
Required for InstallBehavior optional
Available for macOS, iOS, iPadOS

## apps and books for organizations
replaces contentMetadataLookup server api
Customization, versioning, performance, uptime
Visit developer.apple.com
# Secure devices

Manage behavior
Ensure consistency
Ensure compliance
Quick changes
Configure and monitor

macOS has configurable services
sshd - /etc/ssh
Secured for consistency and compliance
Prevent tampering

**Declarative compliance and monitoring**

Configurations to manage system services
Status to monitor background tasks
Predicates power-autonomous behavior

Configuration: `com.apple.configuration.services.configuration-files`
ZIP Archive data asset
Secure service-specific location for files
`libmanagedConfigurationFiels.dylib` to discover file location

Built-in services
Third-party services

built-in services include
* sshd
* sudo
* PAM
* CUPS
* Apache httpd
* bash, zsh

background tasks on macOS
status item: services.background-task
Monitor tasks to verify compliance

uid, state, type, , etc.

## monitor filefault on macOS
`diskmanagement.filevault.enabled`.
Monitor filevault state.  
can be used as activation predicate so that secure things only install if filevault enabled

# Install credentials

Secure network access
Certificates and identities
Manage multiple services
Certificate updates
PassKeys instead of passwords

MDM: 
certificate and identity payloads
ACME and SCEP
Polling certificate list

MDM cannot reference between different profiles, only single one
so you either have to use one profile or duplicate certs between them, etc.
Updates reinstall all payloads

introducing **declarative certificates and identities**

* assets for certificate and identities
* configurations to reference assets

certificate and identity assets
asset: `credential.certificate`
PEM or DER
identity use pkcs#12, or acme/scep.  
Status: `security.certificate.list`.

Deploy passkeys at work

Configuration to provision passkeys
`com.apple.configuration.security.passkey.attestation`
References an identity asset.  Identity attests enterprise passkeys.

Available for macOS, iOS, iPadOS.
[[Deploy passkeys at work]]

S/MIME for accounts
now supports mail and exchange
Use identity assets for signing and encryption
Avialable on iOS, iPadOS


# Transition profiles
Allows you to transition your products over time.  Declarative management in MDM, as simple as sending MDM declarative management command, and syncing over a set of declarations to get activated on device.

Server listens for incoming status reports.  To make this transition easy, a legacy profile configuration was created to allow existing MDM profiles to be set as configuration.

1.  Remove MDM profile
2. Sending DeclarativeManagement configuration that was the same thing.

This can be a disruptive process.  Or leaving a management gap.  MDM developers ahve asked for an easier way to transition.  Here's how.

Now can take over management of installed MDM profiles without removing them.  Server has to send/activate a configuration that contians the same profile as one already instaleld by MDM.  Declarative device management system will take over, without reinstalling or updating it.

Now the profile is owned by configuration, MDM will not be able to make changes
no disruption to device state
no management gap
transition from MDM to declarative device management
available on all platforms


# Wrap up
Building out management features
your feedback is welcomed
Implement now!!


[[23/What's new in managing apple devices|What's new in managing apple devices]]

schema on github



# Resources
* https://developer.apple.com/documentation/devicemanagement
* http://github.com/apple/device-management
