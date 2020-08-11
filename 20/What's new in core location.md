#location
accuracy, app clips, and widgets, oh my

# Location authorization
never, ask next time, while in use, always

Are there any other controls we can give the user?

What does the app need to know about me in order to do its job?
Does it need to know *exactly* where I am?

Sharing just a ltitle bit of location data may make sense.

New prompt for precise vs non-precise.  User has 2 dimensions to specify location information.

there's 'what circumstances', and then 'how much'.

Apps *cannot opt out of this new feature*.  **The user controls authorization**.

`CLLocationManager.authorizationStatus`

Delegate method - deprecated old method.  Note that you need to check both ivars on the manager.

## Delivery
CLLocation gets new locations, with different values for accuracy (high error basically)
recomputed approximately 4 times per hour

[[designing for location privacy]]

Consider placing a banner in your UI to reflect the user's reduced accuracy. I guess they populate this from the ivar.

## What do you do if you only have approx accuracy, but a feature needs exact?

You can send them into settings.  This may not be right?
Temporary grant via core location api.

CLLocationManager allows you to prompt `requestTemporaryFullAccuracyAuthorization`.  

Note each api use needs a `purposeKey` which maps to a string in `Info.plist`.

Your user will be prompted for an accuracy upgrade the next time they launch their app.

# Takeaways

* account for reduced accuracy in app features
* Ask users for full accuracy only when needed
* Use temporary accuracy upgrade when possible

# Background location updates

## Significant location changes

Driven by reduced accuracy locations
Same de-duplication logic applies (if your position hasn't changed enough to change reduce accuracy location)

 ## Visits
 
 Entry/exit times are accurate; only the location information itself is reduced accuracy.
 
 ## Beacons/regions
 
 Beacon and other region monitoring is disabled under `.reducedAccuracy`
 
 Reminders – prompts me to go to settings.  It's correct to send the user to settings here rather than use a temporary upgrade.
 
 User can still technically use this feature, but it may not be any good.
 

 # Reduced accuracy by default
 
 Even though you might be authorized for full accuracy, you might not need them.
 You can set `desiredAccuracy` to `reducedAccuracy` to accomplish this.
 Also can set in an info plist key
 
  In some cases, the user may want to go back and change their mind.  User can toggle on the switch even if the info.plist key is set.
 
 
 
## Case study - TV blacklists

TV only asks for approximate location.  That's all it needs for broadcast rules.

# desiredAccuracy
I thought this was working all along?

First, there wasn't actually a gauarentee you would get the accuracy than you asked for.  We could opportunistically deliver better accuracy if available.

Second, the user had no control over how much information an app would get?

## How is this implemented?

You might think this works by taking your true location, and then adding noise.  However this is **not the case**.

Simply adding random noise on top of location  is not that secure.

It's better to think about this as a quantization.  Reported positions will *contain* the user's true position.

Location snaps to different quantized regions of varying sizes.  In denser urban areas, quantization can be a few km.  In rural areas, it can be 10km or a bit more.  Typically, it's about 5km.  

Reduced accuracy boxes are recomputed about 4x per hour.  So depending on how fast these cars are moving, there may be some lag with these snaps.  The actual semantics of quanitzation are intended to be what the user would expect if they said "where am I?"

We use quantization to adjust the scale to e.g. use smaller scales near a boder, so that you get them on the correct side of it.  However, keep in mind that the center of the region is not where the user is – it's just the center of a quantized area that contains them.

Snapping might put you in a neighboring city, especially if those cities are close together.  But we do try to quantize the user into regiosn on the mexican side of the border.

[[What's new in Core Location]] – share with watchos

If the user selects 'allow once' or does an accuracy upgrade on the watch app, phone will benefit from that decision.

# Some workflow change to state diagram
some authorization flow change taht I did not completely understand.  Evidently this happened on 13.3 or so?

# CLLocationManager.activityType
 
`airbone` - only use for flying
`fitness` - only use for workout session.
`automotiveNavigation` - clips to roads
`otherNavigation` - navigation that does not clip to roads.

# App clips & widgets
[[exploring app clips]]

app clips – will not receive always authorization
while using -> "while using until tomorrow" – automatically cleared.

Don't put any features into an app clip that requires always, because you can't get it.

## WidgetKit

Widgets taht use location must include `NSWidgetWantsLocation` in the Info.plist
Widgets can't show authorization prompts
authorization comes from parent app

consider how to prompt in the parent app should you need it.
