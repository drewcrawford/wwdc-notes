#security 
# Authentication today
Username/password.

We've learned that with great convenience, somes at a cost to security.

* Protecting secrets is hard
* Phishing is very common
* Weak/reused passwrods make problems worse

80% of breaches involve weak or stolen credentials


# Authentication methods
* Memorized passwords
* Password managers
* Password + OTP
* Federated auth

[[Secure login with iCloud Keychain verification codes]]

All these optiosn are

* Easy to use
* Works on all your devices
* Works on non-apple devices
* Always with you
* Security level

But the security level isn't strong and unique.

* manager => only as strong as passwords and additional factors
* OTP => Still are typable, fishable, shared secrets
* If passwords are in your head, you can forget them.  So apps/websites need a separate recovery flow.  This can reduce account security level to the email provider (not something you can control)
* No phishing resistance.  Managers can provide hints, but they still can't prevent someone from manually entering the password themselves.
* All methods rely on shared secrets between user and the server, making them no stronger than the weakest protection of the shared secret.

# Properties of a password solution
* Secure by design
* Easy to use
* Works everywhere
* Recoverable

# Security keys
* WebAuthn standard
* Generally easy to use
* More secure than passwords
* Usable on a wide variety of devices

Safari have supported USB, NFC, lightning for awhile.  Most security keys also support more than one connection method where a single hardware key can be used on different devices

# WebAuthn vs passwords

Passwords: both client and server are responsible for protecting the secret.

Keypairs: asymmetric encryption explanation.  Only device is responsible for protecting the private key, challenge/response.  Server verifies without learning private key.

* Public/private keypairs
* Trust comes from the browser and OS
* No server-side secrets

Less work on the server-side to keep secrets safe.

Servers are less valuable for targets for attackers.

Problems
* Require modern device
* Not always with you
* Not recoverable

In iOS 14.5, we extended security key support to work on all browsers

In iOS 14.5?
* Available for all macOS and iOS apps
* part of the ASAuthorizaiton API family
* Useful in especially high-security contexts


# Passkeys in iCloud Keychain
Technology preview
* Powered by Webauthn
* Backed by iCloud Keychain
	* E2E encrypted
* Very easy to use
* Stronger than post password + 2fa used today

Non-apple devices?

> That functionality is not present in macOS Monterey and iOS 15

Recoverable (iCloud keychain?)

Credentials also work on the web.  

Needs associated domains with `webcredentials` type.


[[Introducing Password AutoFill for Apps - 17]]

1.  `ASAuthorizationPlatformPublicKeyCredentialProvider(domain)`
2.  Create request
3.  pass request to `ASAuthorizationController`
4.  set controller properties
5.  `controller.performRequests()`

Signing in,

`createCredentialAssertionRequest`
similar to create account example

Note that `authorizaitonRequests` is an array.  Can pass in various stuff like signin with apple, etc.

Only caveat is that public key registration requests can't be mixed with non-registration options.

Delegate callback `authorization.credential`.  Two cases: one for registration and another for assertion.  Send the values off to your server, verify them, and finish the operation.

> Technology preview

This is **off by default**.  Enable in developer settings.  

On macOS, check in Safari.  (Develop menu.)

* Off by default
* Only use for testing
* passkey requests are silent when off

# Next steps
* Check out developer documentation
* Adopt WebAuthn on your server
* Try out passkeys in iCloud Keychain
* Give us feedback


https://developer.apple.com/documentation/authenticationservices/connecting_to_a_service_with_passkeys
https://developer.apple.com/documentation/authenticationservices/public-private_key_authentication/supporting_security_key_authentication_using_physical_keys
https://developer.apple.com/documentation/authenticationservices/public-private_key_authentication/supporting_platform_authentication_using_icloud_keychain
https://enterprise.verizon.com/resources/reports/dbir/
