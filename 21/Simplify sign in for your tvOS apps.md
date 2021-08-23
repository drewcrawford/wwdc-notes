#tvOS 

Most common ways to sign in is via password.  Users encouraged to have strong or unique passwords, but they can be frustrating to type.

Easier way for password-based signin.   Use iPhone/iPad to complete signin.

User will see notification on iOS from apple tV.  iPhone will guide them through process of signing in, suggesting iCloud keychain credential, etc.

First-class signin experience where both the appleTV and iphone/ipad cooperate.

# Set up associated domains
Secure association between app and domain
Allows the apple TV and the iPhone/iPad to safely suggest credentials.

1.  Ensure apple-app-side-association lists the tvOS app identifier
2.  Add associated domain capability to tvOS app in xcode
3.  Add domain to the associated domains capability wiht the `webcredentials` prefix

[[Introducing Password AutoFill for Apps - 17]]

# Request a credential
```swift
let controller = ASAuthorizationController(authorizationRequests: [
    ASAuthorizationPasswordProvider().createRequest()
])

controller.delegate = self
controller.performRequests()
```

If your app also supports signin with apple, you may include an apple id request.  When you specify multiple requests, iPhone/iPad will let the user decide which type of credential to sign in.

# Customize the UI
`.other` navigate directly to your own UI.  
`.videoSubscriberAccount` sign in with TV subscriber
`.restorePurchase` allows user to sign in by restoring an in-app purchase

When the user selects one of these methods, up to you to begin the requested signin flow.

```swift
controller.customAuthorizationMethods = [
    // Sign in Manually
    .other,
    // Restore Purchase
    .restorePurchase
]
```

When user selects a custom method, system will call `didCompleteWithCustomMethod`.  Perform the type of signin the user requested.

# Best practices
* Start with single signin button
* Offer a limited number of clear choices
* We recommend system signin view

* Build great sign-in experiences
* Use iPhone or iPad to sign in
* New AuthenticationServices API

[[What's new in authentication - 19]]

