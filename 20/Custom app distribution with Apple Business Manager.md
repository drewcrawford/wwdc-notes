# Custom app distribution with Apple Business Manager
Multiple ways to distribute apps to customers

| App types              | Enterprise       | App Store                  | Custom               |
|------------------------|------------------|----------------------------|----------------------|
| Purchaser              | Own organization | end user                   | Organization         |
| Audience               | Internal only    | General public             | Business or Internal |
| Customization          | Yes              | Everyone gets the same app | Yes                  |
| App Store Distribution | No               | Yes                        | Yes                  |
| App Review             | No               | Yes                        | Yes                  |

Custom is now the preferred path for internal deployments.

# Custom apps
* Private distribution
* Custom features
* Apple Business Manager
* Apple School Manager (new)
* Distribute to customers or internally

Customers purchase a custom app via ABM or ASM.  You as a developer explicitly allow which organizations will see your custom apps and they show up in a separate collection in ABM.

Once the license is distributed to the device, device downloads the app.

# Developer
* Private distribution
* App Store
* App Store Connect
* TestFlight
* Supports internal deployment
* Distribution certificates don’t expire
## Developer team setup
Requires organization developer account
Use the same program for your public apps
Be aware that you need an organization developer program, requiring a duns number.
Invite third-parties to contribute
Manage privileges for external contributors

## App Store distribution
Provide metadata
Set price and availability
Distribute to Business and Education

[[What’s new in App Store Connect]]

## App review guidelines
* Provide demo credentials or demo mode
* Ensure accurate metadata and screenshots
* Add review notes
* Be mindful of App Store schedule
* Use Documented APIs
* Adhere to privacy guidelines
## Maintaining multiple apps
Keep instances to minimum
App Configuration
User authentication
Collect common code into frameworks
New instance requires a new Bundle ID

## Development timeline
Once you publish a new version of your app, your customers cannot deploy a previous versions.

Make sure customers have enabled custom apps
Verify organization name and ID match
Take a breath, wait a minute.  Apps take awhile to show up.  24 hours?
Create a new bundle ID for your custom app
Can’t convert an existing consumer app into custom app, or vice versa
Contact us for help.
Can submit a DTS case.

# Administrator
## Business advantages
* MDM managed deployment
* Flexible license management
* Trusted App Review process
* Customized branding, features

## ABM
* Zero-touch for managed device and apps
* Federated authentication
* User enrollment

[[What’s new in Managing Apple Devices]]

1.  Enable custom apps in settings
2. Provide organization details to developer.  Name/ID must match exactly.
3. Add MDM server.  We’re using Profile Manager, apple’s reference MDM.  Upload key.
4. Download Token and upload to Profile Manager.

## MDM
* Assigning licenses
* Restrictions for updating
* Removing licenses
* Push update
* Defer OS updates for up to 90 days

Use Locations to delegate license management to specific users.  E.g. can purchase licenses from the main location, and assign to regional office.
Migrating from the legacy VPP program, ABM/ASM provide more flexibility.  Need to plan your migration to ensure apps continue to work.  See support article.
VPP will no longer be available dec 1, 2020

Version management - MDM
User based vs device based.

Make sure your MDM solution supports these features
* Supports custom apps
* Restricts app update
* Force app update
* Defer OS update

# User
## End user advantages
* Zero touch setup
* App catalog
* Manageable iCloud benefits

# Wrap up
Custom Apps are the best method for distributing business apps
Developers can create Custom Apps tailored to their customers’ needs
Internal development teams can self-distribute
ABM/ASM

