# app clips
## app clip invocation
* Safari
* messages
* maps
* nfc tag
* qr code
* location

associated with an invocation URL.  URL may contain launch parameters

* determines app metadata in the card
* passed into appclip to deeplink the user into a specific functionality

## Delivery/betatest
an appclip cannot be packaged separately from a full app.  (Iwonder how yelp works?)
Looks like you can have multiple app clips.  
"Testflight is the only invocation case where we launch app clips without showing the card"

### metadata
Image, title/supbtitle, call to action
link to the app store full app.

Default metadata is required for app clips.
This metadata will only appear in safari and messages.  For many of you, this is all you need to set up.
Configured in app store connect.

How does safari know that your webpage has an appclip?  metatag on the app.
For your customers on iOS 13 and earlier, you can add an app id attribute that will link them to your full app on the app store.

## Advanced clip experiences
Metadata can be customized.  
Or associated on apple maps.
Advanced experiences support all invocation methods.

### setting up in app store connect

## associated domains
"build details" section - "view status" shows what apple thinks of your associated domains.

"Load debug status" - We'll reach out to your servers in real-time and validate the app clip portions of your file.

[[what's new in universal links]]

# Game center
Recurring leaderboards - collect scores for a predetermined period of time, which repeats at specific intervals.

# Subscriptions
Family sharing for subscriptions will be available to all app developers

Existing subscribers will need to take an action to optin.

Once you confirm family sharing, you cannot turn it off again.

[[what's new in in-app purchases]]
[[introducing storekit testing in xcode]]
[[architecting for subscriptions]]

# app store connect api
## app metadata api
Create new appstore versions
set pricing,
edit app/version metadata
associate builds with versions
submit to app review

## power and performace metrix and diagnostics api

[[app store connect api]]




