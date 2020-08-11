#appclips 
# Best practices
* focus on essential tasks
* Make your app clip usable right away
* Ask users to sign up after finishing their task
* Only ask for permissions when needed
* Provide the same streamlined experience in your app, because the appclip will be replaced

[[Design Great App Clips]]

Keep in mind the smaller your app clip is, the faster it gets to your users.
For assets that are used in both app and appclip, put in a shared asset catalog.

## authentication
* support Sign in with Apple
* Use `ASWebAuthenticationSession` for other federated login
* Offer username and password login for existing users
* Offer Sign in with Apple upgrade in the app

[[What's new in authentication - 19]]
[[Get the most out of Sign in with Apple]]

## privacy
Some sensitive user data is not available in app clips.
You can always encourage users to get your app.
We transfer permission data when user gets your app.
[[Build Trust Through Better Privacy]]

# Streamlining transactions
1.  NFC
2.  Ask for location access
3.  Order
4.  Pay
5.  Ask to send notifications
6.  Sign in

How to improve this?

First, your app can be triggered by NFC instead of needing location permission.  So we can show the right store based on that.
Or for fraud permission, consider location confirmation instead.
`NSAppClipRequestLocationConfirmation`
```swift
import AppClip

guard let payload = userActivity.appClipActivationPayload else {
    return
}

let region = CLCircularRegion(center: CLLocationCoordinate2D(latitude: 37.3298193,        
    longitude: -122.0071671), radius: 100, identifier: "apple_park")

payload.confirmAcquired(in: region) { (inRegion, error) in

}
```

## payment
Apple pay allows people to make purchases fast and secure.
## notifications
Need to request permission.  You get permission for 8 hours.
`NSAppClipRequestEphemeralUserNotification`.
To tell if a user has provided information in the app clip card, check `authorizationStatus==.ephemeral`

```swift
import UserNotifications

let center = UNUserNotificationCenter.current()

center.getNotificationSettings { (settings) in
   if settings.authorizationStatus == .ephemeral {
        // User has already granted ephemeral notification.
    }

}
```

## sign in

# Transition users to your app
iOS provid