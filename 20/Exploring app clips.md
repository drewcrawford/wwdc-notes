#appclips

# introduction
You'll need an app.
App clip experience -> URL
App clip -> on-demand binary

## Experiences
* entry points into your app
* Invokved by URLs registered in app store connect
* [[Configure and link your app clips]]
* Surfaced through user action and proactively as Siri Suggestions

## Clips
* Second application target
* Part of a single submission for app store review
* Downlaoded separately, mutually exclusive on-device
* Include only what is needed for a fast download
* <10mb after thinning

# choosing the right experience
encountered in-context, and in the moment someone needs them
staple app features may no longer make sense.
consider how URLs can be deep links into your flows

# Xcode
Add target

swiftui - `.environmentObject()` modifier.

# Tech overview
Note that some permissions APIs always return false.
No `isAppClip` API.  "Follow the best practices for the frameworks you use"
Location confirmation API - [[Streamline your app clip]]
App clip shared data container -> migrate data from app clip to app

* Local storage will be deleted after a period of inactivity
* Not included in iOS backups
* Access to personal information is limited
* Can only be launched by the user, or in response to an app clip URL
* Universal links, document types, and URL schemes are unavailable.  e.g., a custom scheme cannot be used as a callback for an authentication serivce.
* Cannot include bundled extensions


# Device states and transitions
while appclips can store data on the device, treat it more like a temporary cache.
If a user repeatedly uses the app clip experience, the lifetime will be extended and it may never be cleaned up.  So the app could cache data.
When installing the app, iOS will migrate some permissions.  You can migrate data using a group data container.  App clip should store data in the shared container, rather than the app clip container.

We recommend apple pay.  Notifications.  SwiftUI.  `SKOverlay` to offer the full app.

[[what's new in in-app purchases]]

Use `ASAuthorizationController` to easily sign-in to their existing password-based accounts.  [[What's new in authentication - 19]]

