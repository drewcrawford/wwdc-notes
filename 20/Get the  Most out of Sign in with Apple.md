#signinwithapple

Over half of users use private email relay.
[[Introducing sign in with apple - 19]]

# Creating a secure request
```swift
// Configure request, setup delegates and perform authorization request

    @objc func handleAuthorizationButtonPress() {
        let request = ASAuthorizationAppleIDProvider().createRequest()
        request.requestedScopes = [.fullName, .email]
        
		//allow you to verify that the credential you get were the ones
		//you expect
        request.nonce = myNonceString()
        request.state = myStateString()
            
        let controller = ASAuthorizationController(authorizationRequests: [request])
            
        controller.delegate = self
        controller.presentationContextProvider = self
            
        controller.performRequests()
    }
```

## nonce
* opaque blob of data sent as a string in your request.
* Later on you will be able to verify this value
* Returned embedded in the `identityToken`
* Helps prevent replay attacks

## state
* like nonce
* state will be returned in the credential

## private email relay
* team scoped
* Routes emails to an email address verified by Apple
* Users are able to reply to your emails

## response

```swift
//get a credential from an authorization
// ASAuthorizationControllerDelegate

func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
        if let credential = authorization.credential as? ASAuthorizationAppleIDCredential {
            let userIdentifier = credential.user
            let fullName = credential.fullName
            let email = credential.email
            let realUserStatus = credential.realUserStatus
            
			//these properties can be verified
            let state = credential.state
            let identityToken = credential.identityToken
            let authorizationCode = credential.authorizationCode
            
            // Securely store the userIdentifier locally
            self.saveUserIdentifier(userIdentifier)
            
            // Create a session with your server and verify the information
            self.createSession(identityToken: identityToken, authorizationCode: authorizationCode)
    }
}
```

* cache the objects you need locally, because some, e.g. name and email, will only be given to you once?
* Verify the state value of the credential to be the same state value you previously generated.
* Validate the information with Apple, on your server

## `identityToken`
The `identityToken` has many fields.

* `sub` (subject), user identifier that was returned to you in the authorization.
* `nonce`, verify to be thes ame value you generated in your request, helps mitigate replay attacks.
* `email`
* `real_user_status` will be returned the first time a user authorizes your app.
	* `0` unsupported
	* `1` unknown
	* `2` likely real

* contains important authorization values
* verify signature with Apple public key
* Exchange authorization code for new tokens
* Use refresh tokens to confirm the user's status.  ("may verify once a day")

# Credential state changes
```swift
// Getting a credential state
        
        let provider = ASAuthorizationAppleIDProvider()
        
        provider.getCredentialState(forUserID: getStoredUserIdentifier()) { 
                                                        (credentialState, error) in
            switch(credentialState) {
            case .authorized:
                // Sign in with Apple credential Valid
            case .revoked:
                // Sign in with Apple credential Revoked, Sign out
            case .notFound:
                // Credential was not found, fallback to login screen
            case .transferred:
                // Application was recently transferred, refresh User Identifier
            @unknown default:
                break
            }
        }
```

Important that this method runs every time your app is launched or becomes foregrounded.

## transferred
* app ownership changed. 
* User migration needed
* Silent migration (to users)

```swift
// Migrating a user identifier

        let request = ASAuthorizationAppleIDProvider().createRequest()
        request.requestedScopes = [.fullName, .email]
            
        request.user = getStoredUserIdentifier()
        
        request.nonce = myNonceString()
        request.state = myStateString()
            
        let controller = ASAuthorizationController(authorizationRequests: [request])
            
        controller.delegate = self
        controller.presentationContextProvider = self
            
        controller.performRequests()
```
# Server notifications
1.  Register your server
2.  JWT event

* disable/enable email events.
* `consent-revoked`: user decided to stop using their apple id with your app
* `account-delete`: user asked apple to delete their apple id
# Sign in with Apple Button
## SwiftUI support
* easy to create and present
* creates authorization request
* handles response

```swift
// SwiftUI example:

SignInWithAppleButton(.signIn) {
    onRequest: { (request) in
        request.requestedScopes = [.fullName, .email]
        request.nonce = myNonceString()
        request.state = myStateString()
    }
    onCompletion: { (result) in
        switch result {
        case .success(let authorization):
            // Handle Authorization
        case .failure(let error)
            // Handle Failure
        }
    }
}.signInWithAppleButtonStyle(.black)
```

## online button editor
https://appleid.apple.com/signinwithapple/button

* label
* size
* position
* download assets

# Upgrading to Sign in with Apple
* secure
* reduce complexity
* Prevent duplicate accounts

---

* fast, easy, security upgrade
* Extension based
* Old credentials are removed
* Supports additional UI
* Upgrade password

[[One-tap account security upgrades]]

## three flows for upgrades
* security recommendations (weak credential)
* password autofill (weak credential)
* within your app

## `ASAccountAuthenticationModificationViewController`
* VC
* non-user interactive (first call).  We hope most upgrades can occur here.
* Step-up security UI, if needed
* Extension context

## extension context
* property of `ASACcountAuthenticationMOdificationViewController`
* Authorization.  Will display UI specific to your app
* Request UI to display
* Tell OS we're complete
* Tell OS to cancel

## implementation details
1.  OS starts flow
2.  Extension initialized
3.  `convertWithoutUI` called
4.  Validate passed credential
5.  Suppose verification requires call to server to confirm credential
6.  Server verifies credential and replies to extension
7.  extension requests apple credential

what if there's an issue?

```swift
enum VerificationResult : Int { case success; case failure; case twoFactorAuthRequired;

override func convertAccountToSignInWithAppleWithoutUserInteraction(
    for serviceIdentifier: ASCredentialServiceIdentifier, 
    existingCredential: ASPasswordCredential
) {
    verifyCredential(existingCredential) { (result: VerificationResult) in
        switch result {
        case .failure: //e.g., server says no
            self.extensionContext.cancelRequest(withError: 
                ASExtensionError(.failed))
        case .success: //e.g., account can be converted now
        	self.extensionContext.getSignInWithAppleAuthorizationWithState(state: myStateString(),
                                                                         nonce: myNonceString(),      
                                                                         {…}        
        case .twoFactorAuthRequired: //need to show UI to user
            self.extensionContext.cancelRequest(withError: 
                ASExtensionError(.userInteractionRequired))
    }
}
```

**We recommend requesting UI only if needed.**  We'd expect in most cases a non-userinteractive flow is enough to complete the conversion.  

1.  Extension cancels with `.userInteractionRequired` reason.
2.  OS tries again in an interactive mode.
3.  VC does whatever.'

## finishing the upgrade
1.  Extension generates state/nonce
2.  gets credential
3.  Extension requests upgrade of the account to the server
4.  backend exchanges authorization code with apple servers
5.  If the server's operation failed, the extension should cancel with an error
6.  otherwise, extension calls complete.
7.  password manager will remove existing credential

Because we remove the old credential, users will not be confused.

* improve user security
* avoid confusion through duplicate accounts
* "convenient API" 🧐




