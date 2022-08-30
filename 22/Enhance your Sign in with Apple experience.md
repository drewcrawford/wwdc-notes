How you can enhance your Sign in with apple experience
* one-tap account setup
* built in security
* verified email address
* anti-fraud

works everywhere, including your managed apple IDs that you use for work and school

[[Discover sign in with Apple at Work & School]]

How you can enhance and streamline your experience
# Prevent duplicate accounts

Convenient and secure alternative to traditional auth.  But users oculd still have accounts that are unlocked with passwords.  Don't create a second account for your app.

Be sure to implement passwor autofill so that existing credentials can appear.

Upgrade to sign in with apple.  Once upgraded, your users will get an account that has security built in, and one less password to remember
* Account authentication modification extension
Fast, easy, security upgrade

[[Get the Most out of Sign in with Apple]]

Present existing credentials as soon as app launches.  Can sign in with the right account even before they reach the login screen.  AS API allows the user to create a credential, API can present existing credentials including password-based credentials.

```swift
// Requesting both Sign in with Apple and password-based accounts.

import AuthenticationServices

let controller = ASAuthorizationController(authorizationRequests: [
    ASAuthorizationAppleIDProvider().createRequest(),
    ASAuthorizationPasswordProvider().createRequest()
])

controller.delegate = self
controller.presentationContextProvider = self

if #available(iOS 16.0, *) {
    controller.performRequests(options: .preferImmediatelyAvailableCredentials)
} else {
    controller.performRequests()
}
```

Start by creating an instance of the authorization controller.    Set delegates.`performRequests` with `.preferImmediatelyAvailableCredentials`.  new on iOS 16, it tells the system that you only want credentials that are immediately on device, called on app launch.  To support previous versions, can use `.performREquests`.  You will be presented with a list of existing ccredentials.  User can select either an existing signin credential, or an existing password credential.

System will call
```swift
// ASAuthorizationControllerDelegate

func authorizationController(controller: ASAuthorizationController,
                             didCompleteWithAuthorization authorization: ASAuthorization) {
    switch authorization.credential {  
		case let appleIDCredential as ASAuthorizationAppleIDCredential:
        // Sign the user in with Apple ID credential.
        // ...

    case let passwordCredential as ASPasswordCredential:
       // Sign the user in with password credential
       // ...
   }
}

func authorizationController(controller: ASAuthorizationController, 
   didCompleteWithError error: Error) {
    // No credential found. Fall back to login UI.
}
```

same auth services API also works for passkeys.  To learn more, [[Meet passkeys]]

With justa f ew lines of code, you acn tak efull advantage of signin experience.  Help users select the right account, etc.  

# Demystify apple ID credential
## Sign in response
After a successful authorization, you get ASAuthorizationAppleIDCredential.  `user`, fullname, email, realUserStatus, identitytoken, authorizationCode, etc.

User => unique and stable identifier.  SAme across all apps in your team.  Uniquely identify a user in your system.  
Name =>Users can share any name they want. 
Email => If you want to communicate with users, ask for email.  Users can use hide my email.  Hidden email address that routes to their inbox.  Two-way relay so it can handle replies.  Email address has been previously verified by apple.  **Not all accounts have an associated email.  be prepared to handle this.**
realUserStatus => calculated using on-device ML, account history, and hardware attestation.  likely real: user appears to be a real person.  unknown => system hasn't determined.  Limited information.  Unsupported => "Not capable of this determination."

Be aware that fullName, email, and realUserStatus are only available **when an account is created.**  Not returned for future signins.  Securely cache these until you can verify that an account has been created in your system.

`identityToken` => JSOn web token.  Industry-standard approach to authentication.

JWT has various parts
* header
* payload
* signature

Verify the signature with apple's public key.  Also check the validity of the token.  Once you decode the payload, you should verify the issuer is `appleiid.apple.com`, the audience field is your bundle ID.  Make sure the epxiry timestamp is greater than the current time.  Subject will be your user identifier.  If you requested the user's email address, that will be included.  The value will be 0 for unsuported, 1 for unknown, 2, for likely real.

nonce.  [[Get the Most out of Sign in with Apple]]

`authorizationCode` => short lived, single-use token.  Provide in exchange for a refresh token.  Might be familiar from OAuth2.  Send a POST to appleid.apple.com/auth/token.  client id, client secreet, and the authorization code.  Detailed description in docs.

In the response, you get a refresh token, access token, and an identity token similar to what you got earlier.  You can use the refresh token to obtain a new access token.  Continue to use the same refresh token until it is invalidated.  If token verification fails or if there are changes around the user session.

# Handle session changes
After verifying the token, your app is responsible for managing the user session.
A user can stop using apple ID with your app on settings.  Or sign out of the device.  So check credentials on app launch.

```swift
// Check User Credentials on app launch

let appleIDProvider = ASAuthorizationAppleIDProvider()
appleIDProvider.getCredentialState(forUserID: "currentUserIdentifier") 
{ (credentialState, error) in
    switch(credentialState){
    case .authorized:
        // Found valid Apple ID credential
    case .revoked:
        // Apple ID credential revoked. Log the user out.
    case .notFound:
        // No credential found. Show login UI.
    case .transferred:
        // Team is transferred
    }
}
```

Also observe for notifications.

```swift
// Register for revocation notification

let notificationName = ASAuthorizationAppleIDProvider.credentialRevokedNotification

NotificationCenter.default.addObserver(self, 
                                       selector: #selector(signOut(_:)),
                                       name: notificationName, 
                                       object: nil)
```

A different user may have signed in.  If you have an app server, should subscribe to server-to-server notifications.  Receive important updates about users, etc.

1.  Disables or enables their mail forwarding preferences
2. stops using apple id with your app
3. user permanently deletes their apple ID

Register an endpoint URL in dev portal.    Events sent as JWT signed by apple.  `email-disabled` for forwarding disabled.  `consent-revoked` for stop using with your app.  Invalidate any active user session when you get this event.  If the user deletes their apple ID, you will get an `account-delete` event.

## Account deletion
We use them to manage our private data.  Someone might want to delete their account, support in your app.  Provide a way to initiate account deletion, it is your eresponsibility to manage the process.

If you have an app server, typically the app notifies the server to delete user accounts.  New REST endpoint that your server can use to delete an account on apple's side.

1.  Must have a valid refresh token or valid access token.  If you don't have either, can generate using `/auth/token`
2. Use the `/auth/revoke` endpoing with required parameters.  

If you get a successful repsonse, tokens and sessions are instantly invalidated.  user returning to your app will have an experience similar to when they first created the account.
# Implement across platforms
How to group similar apps together.  Group similar apps to streamline the user experience.  Users only need to provide their consent once to share information with your app.  

## Configure a services ID.
Log into dev portal.  Navigate to cert, identifiers, and profiles.

Services IDs.  Continue.  Click on checkbox next to sign in with apple and then check the configure button.  Select a primary app ID from the dropdown.  Input the domains and subdomains your website will use to support sign in with apple.  Enter a redirect URL for apple to redirect your user back to your app or website.

## Add sign in with apple button

Highly configurable button API.  Use this to customize and embed in your button of choice and embed in your app or website.

Integrating sign in with apple even easier on the web.  In your app or website, start by including in your JS framework.  This simple API will allow you to authenticate your users, etc. 


```html
// Embed Sign in with Apple JS
<html>
    <body>
        <script type="text/javascript" src="https://appleid.cdn-apple.com/appleauth/static/jsapi/appleid/1/en_US/appleid.auth.js"></script>
        <div id="appleid-signin" data-color="white" data-border="true" data-type="sign in"/>
        <script type="text/javascript">
            AppleID.auth.init({
                clientId : '[CLIENT_ID]',
                scope : '[SCOPES]',
                redirectURI : '[REDIRECT_URI]',
                state : '[STATE]',
                nonce : '[NONCE]',
                usePopup : true
            });
        </script>
    </body>
</html>
```

change colors
```html
<div id="appleid-signin" data-color="white" data-border="true" data-type="sign in"/>
```

```html
<div id="appleid-signin" data-color="black" data-border="true" data-type="sign in"/>
```

change button text to continue
```html
<div id="appleid-signin" data-color="black" data-border="true" data-type="continue"/>
```

create logo only button

```html
<div id="appleid-signin" data-color="black" data-border="true" data-mode="logo-only"/>
```

see resources for addtiional option.

Can generatea  button with REST endpoint.  Separate endpoints for center-aligned, left-aligned, or logo button.  Customize the button using query parameters.   Response as PNG image.

## Send authorization request
Send required parameters to apple


```html
// Embed Sign in with Apple JS
<html>
    <body>
        <script type="text/javascript" src="https://appleid.cdn-apple.com/appleauth/static/jsapi/appleid/1/en_US/appleid.auth.js"></script>
        <div id="appleid-signin" data-color="white" data-border="true" data-type="sign in"/>
        <script type="text/javascript">
            AppleID.auth.init({
                clientId : '[CLIENT_ID]',
                scope : '[SCOPES]',
                redirectURI : '[REDIRECT_URI]',
                state : '[STATE]',
                nonce : '[NONCE]',
                usePopup : true
            });
        </script>
    </body>
</html>
```

> Since you've already implemented on an apple platform, these parameters are very familiar.

If your app or website requires email or name, fillin the scope parameter.  Use a space to separate each scope.  Improtant that you only request the data you need.

Redirect URI lets you add URL you've registered previously.  Informs apple where to direct the user to on your website.  Add a state and NONCE to secure your request.

usePOpup parameter to display the login screen in a separate login window, or have an existing window redirect.  If someone is using safari, you'll see a native screen like this one, provide them a first-class experience

## Handle response
To handle a success response, add an event listener.

```html
// Listen for authorization success.
document.addEventListener('AppleIDSignInOnSuccess', (event) => {
    // Handle successful response.
    console.log(event.detail.data);
});

// Listen for authorization failures.
document.addEventListener('AppleIDSignInOnFailure', (event) => {
     // Handle error.
     console.log(event.detail.error);
});
```

you'll receive a response with autohrization code, identity token, and information they requested.  Similar to response on apple platforms.  If you'd prefer to use a rest api, you can do that as well

# Wrap up
Keep in mind
* defer account setup until necessary
	* ex, you may allow a user to purchase an item using apple pay, and then tie purchase to an account later
* offer existing users accuont upgrades
* Respect user chosen email address
	* Don't prompt for additional email
* Implement across all platforms


* https://developer.apple.com/documentation/sign_in_with_apple/revoke_tokens
* https://appleid.apple.com/signinwithapple/button
* https://developer.apple.com/documentation/authenticationservices/implementing_user_authentication_with_sign_in_with_apple