Create, manage, grow apps.  We bring new features to ASC on web, iOS/iPadOS, and API.

let me quickly highlight some recent updates.
TF - associated tester groups easier.  With one click, you can quickly add/rmemove a tester group.
enhanced appstore submission experience.

# Enhanced submission experience
* multiple items in one submission
	* Since review items can only be submitted as part of a review submisison, we need to add it
	* submission is the vehicle that carries items to app review
	* by grouping several items, they're reviewed in context together
	* efficient review, 24 hours
	* no review items are papro ed until all the items in the review submission are accepted.
	* Can edit any rejected review options and resubmit.  
	* or can remove rejected items, so submission contains only reviewed items
		* Keep in mind that removed items would need to be resubmitted to become paproved.
	* items can be 
		* app versions
		* in-app events
		* custom product pages
		* product page optimization tests
* submit without needing a new app version
	* Each submission has an associated platform, with specific review items

| tvOS               | ios                            | macOS               |
| ------------------ | ------------------------------ | ------------------- |
| app version (tvOS) | app verison (iOS)              | app version (macOS) |
|                    | inapp event                    |                     |
|                    | custom product page            |                     |
|                    | product page optimization test |                     |
Can have one in progress subimssion per platform.  

app review reviews all items int he submission against an app version to ensure it's consistent.  If version is in submission, that's the one we use.  But we can submit without adding a version.  This requires a previously approved version of your app.


* dedicated app review submission page

Click app review link in ASC.  Manage your entire review workflow.  Overview of submissions, click into any of them, etc.  Using web UI is great, but wouldn't it be nice to track submission status on the go?  New submission experience to iOS/iPadOS as well.  You can now submit submissions that are ready already.

Opt into receive timely notifications about updates.  Manage submissions by removing items, viweing rejection reasons, or applying to app review.  Available now on ASC for iPadOS and iOS.

# API
last year: app clips, xcode lcoud, in-app events, product page optimization, and app store submission experience.  

This year: 2.0.  60% more API.  

* In-app ppurchases and subscriptions
* customre reviews and developer responses
* app hang diagnostics

## IAP
* new subscription resource
* create, edit, and delete
* manage pricing
* submit for review
* create offers and promo codes.

## Customer reviews
build great cutsom workflows around customer interactions

## app hang diagnostics
Identify and eliminate hangs.  Increase performance and improve UX.  Until now, you could only view metrics via API.  This summer, we're adding diagnostic signatures.  

Use with existing diagnostic signatures resources.

Download detailed logs.

[[Identify Trends with the Power and Performance API]]
[[Track down hangs with Xcode and on-device detection]]
**We will begin to decomission the XML feed this fall.**

# Wrap up
* streamline app store submission process
* download ASC app.
* Automate wtih App Store Connect API




https://developer.apple.com/app-store-connect/
https://developer.apple.com/documentation/appstoreconnectapi