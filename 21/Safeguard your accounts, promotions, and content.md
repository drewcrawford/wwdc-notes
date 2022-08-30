# Designing for trust and safety
* Promotions
* User ratings
* Customer logging
* Premium content
* Text input

Target for fraud and abuse?

## Accounts
Used to protect access to data.

## Promo
Targeted by attackers seeking discounts or otherwise manipulate app/server.

## Content
Scrape servers for content.  
# Account risks
Problems can arise
* Others can compromise security
* taken over through user phishining, password leaks, misplaced trust

Phishing attacks: doubled over the course of 2020.  Attackers may target accounts.

If you use customer feedback, e.x. ratings or recommendations, you may be creating a situation where that feedback can be used for illicit gain

* Overload infrastructure
* disrupt ux
* impact authenticity

[[Introducing sign in with apple - 19]]
[[Get the Most out of Sign in with Apple]]

Apple 2fa.  Credeintial returned indicates with a high degree of confidence that a user is real and not a bot.  Flag can be checked on the client.

Maybe do additional challenge if the flag is unknown.  

On-device ML.  Server-side inferences based on account activity.

* Encourage strong, unique passwords
* Use available data to detect in authentic accounts

[[One-tap account security upgrades]]
[[AutoFill Everywhere]]

## Detecting inauthentic accounts
* account details
* device details

funnel into ML?

Limit account creations that can occur on a single apple device.  More difficult to create large numbers of accounts.

DeviceCheck can help.  


# Promotion abuse
Suppose you want to offer an in-app promotion to grow your business.

Some people abuse the promotion to redownload the app.

DeviceCheck can limit number of promotion redemptions per device.

Set 2 bits of information per device in a way that's strongly tied to device autenticity

Read back even after complete device reset.

* You control the bit values
* Determine if the device has been used for th epromotion in the past

Set bits for account creation.

[[Privacy and your apps - 17]]
[[Mitigate fraud with App Attest and DeviceCheck]]


# Content risks
An attacker can develop/distribute unauthorized software.  Cheating, deception.

## App Attest
Provides app protection.  Enables you to verify that the app connecting to your server are the ones you published.

Leveraging secure enclave, iOS/tvOS generate a cryptographic attestation from app identity.

Server verification before making access available.  

* Create and attest key
* Generate assertions
* Verify app identity on the server

[[Mitigate fraud with App Attest and DeviceCheck]]

# It's your turn
* sign in with apple
* devicecheck
* app attest to validate authenticity

