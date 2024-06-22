At Apple, we believe privacy is a fundamental human right. Learn about new and improved permission flows and other features that manage data in a privacy-preserving way, so that you can focus on creating great app experiences.

Privacy pillars
* data minimization - minimum amount of data needed
* on-device processing - perform computation locally on device
* transparency and control - honoring a user's autonomy by transparency over data collection and control of how it's used
* security protections - foundation for privacy protections.  

# New pickers

Out-of-process pickers.  Information can be selected and just that piece of data is sahred with the app.

existing:
* contacts
* photos
* journaling suggestions

new:
* financekit - transaction data, etc.
* image playground
* accessorysetupkit



## financekit
* transaction picker -> specific transactions
* ongoing access control.  App requests access to all financial data.  Requires entitlement.

[[Meet FinanceKit]]

## image playground
present the sheet.  

## accessorysetupkit

several types of permission into one out of process picker.  Bluetooth/wifi, new accessory management tools.  Combines all required permissions into one straightforward flow.

bluetooth pairing prompt, wifi join prompt, and bluetooth accessory pairing confirmation.

accessory naming
accessory assets
accessory cleanup

[[Meet AccessorySetupKit]]



# Upgraded protections
## Private Wifi

MAC address tracking risk.

private wifi addresses now rotate on iOS, and macOS too.
mac address will change approximately every 2 weeks.  

for public networks, wedefault to a static rotating address.

## macOS extensions

New extension control.  We now generate many more notifications on install

* spotlight importers
* dock tiles
* smart card reader drivers
* color panels
* media extensions
* cron (disalbed by default)

login items and extension now live in one place in macOS system settings.  Easier to understand and control if/when your app is running.  Update your in-app instructions for new location.

Deprecated:
* directory services plugin
* legacy quicklook plugins
* com.apple.loginitems.plist


## app group container protections
sandboxing is good

Now the same protections for a single app can extent to shared containers for a group of multiple apps.

Like app containers, another developer accessing the container gets prompted.

Can also use group containers to store a subset of data, if you're not ready to sandbox your whole app.  Store just that data in a protected group container.  

### best practices
* declare app groups entitlement
* format identifiers correctly
* `containerURL(forSecurityApplicationGroupIdentifier:)` to get path to shared container

see docs
# Permission changes

## contacts
Now two stages
* first, share any data
* second, limited or full access

contact access button.  Instead of a full-screen picker, it blends into your UI, shows contact search result, only requires one tap.

[[Meet the Contact Access Button]]

## bluetooth

shows a map indicating the location of the device.  And sample of associated devices.  

## local network

Now on macOS.  Ensure that your app and its processes only request access in a contextual way.  Poeple are less likely to allow access when prompted without context.

If you have bonjour, or multicast entitlement, you must include the description or else you will be blocked.

* bonjour browsing or advertising
* custom multicast
* custom broadcast
* Unicast connections (local network)

[[Support local network privacy in your app]]

# New platform capabilities

## Locked and hidden apps
* authentication required
*  covers all entry points
* app contents hidden across system


## automatic passkey upgrades

Add support for passkeys
Passkey upgrades are automatic
Works with existing flows

[[Streamline sign-in with passkey upgrades and credential managers]]


## private caller ID
Homomorphic encryption.

Adopting these tools allows you to provide timely information without privacy compromise.

Open source resources available late 2024.

See docs

# Next steps
* use data pickers
* Prepare for improved protections
* Understand new permissions
* Take advantage of new capabilities

# Resources
* https://developer.apple.com/documentation/sms_and_call_reporting/getting_up-to-date_calling_and_blocking_information_for_your_app
