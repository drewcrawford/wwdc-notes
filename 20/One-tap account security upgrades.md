# Account Security
Manage our most personal, private data
Passwords
Authentication services

Accounts should be convenient to use and secure.

# Recommendations
New section of the password manager provides recommendations about why passwords are flagged etc.

* Reused
* Easily guessed
* Compromised (seen in public data breaches)

"Change password on website" button.  "If the service has adopted that standard."
Ideally, it should be faster.  This year, it will be possible to upgrade now.

# Account authentication modification extensions
Provide fast, easy account upgrades
Upgrades to sign in with apple and strong passwords
Can perform additional authentication before upgrading


# extension stuff
## Associating your app with your domain
```js
{
	"webcredentials": {
		...
	}
}
```
[[Introducing Password AutoFill for Apps - 17]]

## Add account authentication modification extension (target)

`ASACcountAuthenticationModificationSupportsStrongPasswordChange` = `YES` in info.plist

This supports both by default ?

```swift
import AuthenticationServices
class UpgradeViewController: ASAccountAuthenticationModificationViewController { }
```

This is a VC because you might decide to display UI.

We call this:

```swift
override func changePasswordWithoutUserInteraction(for serviceIdentifier: existingCredential: newPassword: userInfo:)
```

Can specify user-visible error in `userInfo` when you send the failure

You can modify the password sent by the system if you need to.

You can specify your own password rules
developer.apple.com/password-rules/

## Supporting signin with apple
[[Get the most out of Sign in with Apple]]
[[Introducing sign in with apple - 19]]

Info.plist -> `ASAccountAuthenticationModificationSupportsUpgradeToSignInWithApple`.  By default, the template supports both, set to `NO` if you only support 1.

We call`convertAccountToSignInWithAppleWithoutUserInteraction`.

`userInfo` is passed in is always nil for system-initiated upgrades.

It seems that in this flow, what happens is a bit weird:

1.  First you check if the old credential is ok (like server accepts it etc)
2.  Then you request a sign-in with apple token.  Note that this can fail in some cases, like network-related.  Here you abort the request
3.  Then you actually send the new stuff to the server
4.  Finally you complete the request

Once this is complete, **the system will delete the keychain credential used for the upgrade.**

## Step-up UI
("further authorization is required")

Here in the "authorized" step, you cancel with `userInteractionRequired` error code.  System will create a new request configured to do interaction.

Interface is presented with a system chrome that contains a cancel button.

System will call `.cancelRequest` on your class.  by default, the super implementation will cancel the request.

## In-app upgrades
Your app creates and performs an upgrade request.  The request will cause extension to be launched.

Request will ahve a username, but not a password.  "Your app should not be storing plaintext passwords".  However, you may pass any information with the `userInfo` property of the request object.

```swift
let serviceIdentifier = ASCredentialServiceIdentifer()
let username = ...
let userInfo = [shinyUserAuthTokenKey: ...]
let upgradeToStringPasswordRequest = ASAccountAuthenticationModificationUpgradePasswordToStrongPasswordRequest(user: username, serviceIdentifier: serviceIdentifier, userInfo: userInfo)
let requestController = ASAccountAuthentificationModificationController()
requestController.delegate = self
requestController.presentationContextProvider = self
requestController.perform(upradeTo...)
```

# Best practices

* Make the upgrade experience effortless
* Leverage on-device data to authorize upgrades
* Sign in with apple means no more password
* The cancel button should cancel as soon as possible
* Offer upgrades in App Clip to App transitions

