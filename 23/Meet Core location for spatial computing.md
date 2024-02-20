Discover how Core Location helps your app find its place in the world — literally. We'll share how you can build a spatial computing app that uses a person's location while respecting their privacy. You'll also learn how your app can request location access and how Core Location adapts requests from compatible iPad and iPhone apps.

CL helps you have an anchor in the real world.  How to do just that with CL?

# How to use Core Location

Review the basics.  Sample code.

CLLocationUpdate.liveUpdates()

We invoke the API to do so, etc.  Ensure that we set this string - privacy when in use usage description.

In this app, we ask for location access straightaway,s impel example, etc.

We get a prompt, grant access to private info.  Before moving on, let's see how CL treats user privacy n the system.

# Privacy

* UPdates only with user consent
* Invoke `requestWhenInUseAuthorization`
* Ask only when needed

Users will see a prompt similar to this.  may look familiar to those of you with experience with iOS dev.  User can grab location access for just a session, whiel using the app, or deny entirely.

Users may choose to grant app knowledge of either precise or approximate location, just like on iOS.

[[What's new in core location]]

Location accuracy - similar to a Mac - about 100m.
However, better with iPhone nearby.

For both newcomers and experienced developers, app is eligible while user is using app.

User experience is after all very different from iPhone/watch.

# When is an app in use?

[[What's new in core location - 19]]

* while foregrounded until backgrounded (ex iPhone)
* grace period after entering background

For a fully immersive experience, as long as the user is running the app, we consider it to be in-use and eligible for location updates (assuming consent etc)

window, user "recently looked at the app".  So long as the user is not interacting or looking with eitehr app, neither one will get location updates.  If the user starts looking at (interacting with) the app, then it can get location updates.  

User can move two apps together such that they look at apps at the same time.

Also a grace period while we let it still get updates for awhile.  "A few seconds".

Good experience, respectful of user privacy, etc.

Apps cannot get location updates while not running.  Updates from monitoring APIs will not be delivered.

# Compatibility


"While using" derives from eyes.  
"Always" is treated as "while using".   "Always" is not an option.

* some APIs are unsupported on xrOS.  Region monitoring, CLLocationMonitor, etc.
* What does your app assume?

watch some other sessions.

[[Discover streamlined location updates]] - new developments in API, particularly around how to get location updates, and ways in which we've made our api more compatible with swift ocncurrency

[[Meet Core Location Monitor]] - how we reimagined updates, etc.

* privacy model based on eyes
* consider compatibility behavior


# Resources

* https://developer.apple.com/documentation/corelocation/adopting_live_updates_in_core_location
* https://developer.apple.com/documentation/corelocation
* https://developer.apple.com/documentation/corelocation/monitoring_location_changes_with_core_location
