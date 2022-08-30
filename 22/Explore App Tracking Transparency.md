
Privacy is a human right.  Giving peopel choices and control over how their data is used.  When people have resources and understand ohw their data will be linked or shared, they are more likely to trust and engage with your app.

Last year, policy requires apps to receive the user's permission before trackign them third-party by adopting ATT framework.

# When to adopt
What is tracking?
Tracking refers to linking user or device data collected from your app with user or device data collected with other ocmpanys' apps for targeted advertising or measurement purposes.  Sharing user or device data with data brokers.

Examples.

Suppose I download an app.  Feature that lets me search for places and events that are happening nearby.  I use this for places that serve waffles near me.  It stores an interest of mine.  It shows an ad for people who like waffles.  In this example, it doesn't link my data with any data from an app or website owned by another company.  This scenario is *not* tracking.

Suppose the company has another app that I use.  The server links together info between both apps.  After linking this data, they show me an ad based on what I did in other app.  This is *not* tracking, because it doesn't link my data with any data from an app or website owned by another company.

Food delivery app.  I've used the food delivery appt o palce orders late at night.  Gave it an email address.  Food delivery app includes code that shares my email address.  Shows me an ad based on combined information.  This does require permission to track because it links data with another company's data.  Data is linked together with an email address.

Even if the email address is hashed.  It would *still* require permisison to track, because it is still linking data about a user with another company's data about that user.  Type of data and wehther or not it's being hashed doesn't change anything.

How third party SDKs use and share data from your app.  aS a developer, you're repsonsible for the behavior of your whole app.  Suppose the pal about developer hasn't written any code themselves that would require submission.  But they want to include a 3rd party SDK for advertising purposes.  Whether they need permission depends on whether or not the SDK combines user data from the app with other company's apps or websites.

If the SDK shares user data to provide analytics about ads, but doesn't link user data from other companies, it doesn't require permission to track.  But, if the SDK shares user data with an ad network.  And the ad network links the data it receives with data bout ads I've gotten in other companys' ads.  This requires request permission to track.  Tracking regardless of whether the app or SDK uses data for this purposes.  Or even if you only get aggregate reporting.  If you're unsure aobut whether an SDK would contain code that would require ATT, you should ask the developer of that SDK.  This responsibility is not just SDKs, but any libraries or third party code you use.

Sharing user or device data with data brokers.  What are data brokers?
* sometimes defined by law
* a company that regularly collects/sells/licenses/ to third parties, information about end users whom the business does not have a direct relationsihp.

If you send data to a data broker, that is tracking.  Regardless of whether or not the data broker links with other companies.  Sharing of user data with a data broker requires permission to track.

Even if the pal about client code doesn't directly send account idenfier to the data broker, but instead it sends it to dev server, and that sends it to the data broker.  This would require getting permission to track, even though you're not communicating with the data broker directly.


# How to adopt
Calling the framework.  `requestTrackignAuthorization` method.  CAuse system permission alert to appear over your app.

One-time prompt.  System will remember the user's choice and won't prompt again unless the app is uninstalled.

Provide an `NSUserTrackignUsageDescription` key.  String provided here will be shown in the system prompt that informs the user why the app is requesting.

Dont' include the app name, we identify your app for you.  If you odn't include a usage string, your app will crash.

Use `trackingauthorizationStatus` to determine authorization.  If user has selected allow, then you have their permission.  

Users can change or revoke tracking authorizaton at any time. Make sure your app checks authorization status each time.  Only continue to track when authorized.

Per-app basis.  Just because a user has given one of your app's permission to track, doesn't mean you have permission in another app, even owned by the same company.  Request permission of that particular app.  Apps or websites linked.

If your app doesn't have tracking authorization, a couple of things to keep in mind.

1.  App must not gate functionality
2. IDFA API will return zeros

## Non-tracking alternatives
1.  First party or contextual ads
2. Privacy preserving ad attribution
[[Meet privacy-preserving ad attribution]]
[[What's new in SKAdNetwork]]

Declare data used to track

[[Create your Privacy Nutrition Label]]

## What about fingerprinting
**Using signals from the device to try to identify the device or user is not allowed.**
* web browser properties
* device configuration
* location
* network connection
Collecting any data solely for the purpose of generating a fingerprint is not allowed.


* https://developer.apple.com/app-store/ad-attribution/
* https://developer.apple.com/app-store/user-privacy-and-data-use/
* https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy