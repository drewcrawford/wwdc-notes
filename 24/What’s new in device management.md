

Learn about the latest management capabilities for iOS, iPadOS, macOS, and visionOS, then discover the latest changes to Apple Business Manager and Apple School Manager. We'll also share updates to Activation Lock, SoftwareUpdate, and Safari management.

# Apple Services

## Deployment

with automated device enrollment, ship directly to users, enroll into MDM, customize setup, etc., without having to physically touch devices.

Mac, iPhone, iPad, AppleTV.  This year we're adding vision pro.  visionOS 2.0

device_family-Vision
os=visionOS

Updated setup assistant skip keys.

## devices
both apple vision pro and apple watch can be added to organization at time of purchase.

When a device is wiped/locked, activation lock can prevent unauthorized users can using the device, but it may be left on unintentionally.

Now you can turn off AL for organization-owned devices in ABM/ASM.

* iPhone, iPad, Mac, Apple Watch, Vision Pro
* must be in your organization
* organization and user AL

## identity

Managed accounts.  last year we introduced significant updates, 

Streamlining the domain capture process and ensure all accounts are using your organization's domain

* verify organization owned domains
	* but a personal apple account might be on the domain
* Now you can limit accounts to only managed accounts
* Initiate account capture after verifying domain (without identity provider?)

Users are asked to choose a different email address.  While this frees up account name to be reused, may accounts were created just for work.  This year, users can convert existing account to managed account, adding to organization.

ABM/ASM are critical tools for organizations and these features will make it easier for IT teams to manage their organization's devices.

# Platform updates

Managing software updates.  Keeping devices up to date is an essential component of managing devices in an organizational environment.  Last year we allowed enforce updates with date/time.

This year, software update configuration that replaces all legacy management.  iOS18+ macOS 15+ etc.  

control default notification behavior
manage beta updates

beta updates
* enroll into appleseed for IT
* add the device to the beta program
* Enforce and defer updates
* View status

get available beta programs 
authenticate using oath
program tokens are turned to abm/asm, and then to the mdm.

Offer, enroll, unenroll, block supervised iPhone and iPad devices.  

* set the beta program during enrollment
* 403 status code
* `RequireBetaProgram` which has `Description` and `Token`.  
Remains in the beta program.  

Organization can remotely enroll different devices into different programs, 


## safari management

Define allowed extensions!
`com.apple.configuration.safari.extensions.settings`
control managed extensions (always on/off)
Configure extension website access
works in private browsing

## vision pro management

[[Introducing enterprise APIs for visionOS]]

MDM for visionOS in 1.1.  Best to think of managing vision pro the same as iPhone/iPad.  

in 1.1 we have
* device enrollment
* user enrollment

visionOS 2.0 adds
* automated device enrollment (zero-touch)

Configurations
New MDM commands
New restrictions

visionOS now supports most configurations/payloads including passcode, domains, and web content filter payloads, plus many new MDM commands, like `DeviceConfigured`, etc.

most populate and relevant restrictions, similar to iOS/iPadOS.  

ex allowCamera we adjust screenshots to not show your environment.  

## Mac management

Install IT management tools and other binaries with Services configuration
Deploy launchd configuration files
Secure and tamper resistant

Manage external and network storage
Define mount policy:
* allowed
* disallowed
* readonly
* Replaces media restrictions payload - removed in a future release


Platform single signon. - sync local account credentials with identity provider
* filevault unlock
* Login policies
* Login window
* Lock screen
Stronger security options

Profiles is renamed to device management and has been moved under parity with iPhone/iPad.  Login items and extensions under general as well.

## iPhone/iPad management

eSIM preservation when erasing a device
Manage transferring eSIM to a different device
Setup eSIM with a link or QR code on device
5G network slicing with per-app VPN
Multiple private network support

Locking and hiding apps
* manage ability to lock and hide apps
* Restrict all or specific apps (supervised)
* Per-app basis (managed apps)
* Note: hiding an app also locks it?
* Device enrollments report hidden apps
* User enrollments report hidden managed apps

## stole device protection
* enrolling in mdm
* manually adding an exchange account
* manually installing passcode declarations or exchange payloads

exception for initial case.

Trust in-house apps
Restart required to trust new team identity
In-house apps installed without MDM
One restart per team identity
Identities trusted prior to upgrading will be migrated

# Education enhancements

Easy student sign-in
Classroom nearby ad-hoc classes

## schoolwork
With schoolwork 3.0, we're introducing new assessments and scoring workflows.

Send assessments by scanning/importing existing documents.  Pages, numbers, keynotes, google suite documents, PDFs, etc.

Analyze performance per question.  Analytics, trends, etc.

## assessment mode
Disable certain hardware/software features at launch.  We're now excited to bring multi-app mode for iPad.  ex notepads, spreadsheets, assistive apps, coding apps, and calculator for iPad.

* configuration for MDM
* assessment mode restrictions
* Test-ready for standardized exams

# Wrap up
* transition to managed apple accounts
* Utilize new activation lock features
* Test and manage beta software updates
* Customize safari for your organization
* Begin your apple vision pro deployments


# Resources
* [Apple Business Essentials User Guide](https://support.apple.com/guide/apple-business-essentials/)
* [Apple Configurator User Guide for iPhone](https://support.apple.com/guide/apple-configurator/)
* [Apple Configurator User Guide for Mac](https://support.apple.com/en-gb/guide/apple-configurator-mac/welcome/mac)
* [Apple Platform Deployment](https://support.apple.com/guide/deployment/)
* [Apple School Manager User Guide](https://support.apple.com/guide/apple-school-manager/)
* [Classroom for iPad User Guide](https://support.apple.com/guide/classroom/welcome/ios)
* [Classroom for Mac User Guide](https://support.apple.com/guide/classroom/welcome/mac)
* [Device Management](https://developer.apple.com/documentation/devicemanagement)
* [Device Management Client Schema on GitHub](http://github.com/apple/device-management)
* [Forum: Business & Education](https://developer.apple.com/forums/topics/business-and-education-topic?cid=vf-a-0010)
* [Schoolwork User Guide](https://support.apple.com/guide/schoolwork-teacher/welcome/ios)
* [Support - Apple Platform Deployment](https://support.apple.com/guide/deployment/welcome/)