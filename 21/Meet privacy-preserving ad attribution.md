#privacy 

# Cookies, ads, and tracking
Provide support for ad atribution

Linking click to purchase.  e.g. conversions.

ex cookie data allows shop to measur eeffectiveness of ad campaign.

Companies can use info to follow users across sites and build detailed profiles.  We call this "cross-site tracking".

Most users don't expect to have an invasive experience on the web.

Tracking can break trust and create a misalignment between users and businesses.  Users turn to ad/content blockers.  

How to protect user privacy while maintaining web compatibility?

## ITP - intelligent tracking prevention
First-party cookie from the ad.  Now, ITP prevents cookies and other website data from being sent to third-parties like the store.

ITP mitigates cross-site tracking, but tracking is not unique to the web.  IDFA => similar profiling for iOS users.

## SKAdNetwork
Collects and reports data about ads that lead to app installs.  Privacy-preserving techniques for ad attribution.

## Regulations
CPPA, GDPR, requiring websites to inform users about how their data is being used.

Other browsers – brave, firefox

Business models that rely on tracking aren't sustainable.  More private way to measure ads.


# Measure ads from web to web
## PCM - Private Click Measurement

* Proposed standard
* Processed on-device
* Prevents identification
* No cross-site tracking

1.  Tap ad for longboard
2.  Link specifies source ID: 8 bits.  Destination: site this will convert on.  Stored on-device, in the browser.
3.  Navigated to store.
4.  Add to cart.  Store can specify information called "Trigger data" => 4 bits.  Action like add-to-cart.  Also stored on device in browser.
5.  Only the browser has both halves.  So if the conversion matches a stored click, can format into report.  Schedules to be sent both to source and destination sites, 24-48 hours later.
6.  Delay prevents time-related data from linking together.  These reports also have IP address protection.

[[Apple's privacy pillars in focus]]

How to specify?

```html
<a href="foo"
attributionsourceid="55"
attributiondestination="https://example.biz"
```

on destination
```html
img src="https://example.biz/convert/?"
```

needs to redirect to

```html
https://example.biz/.well-known/private-click-measurement/trigger-attribution/15/[optional 6-bit priority]
```

Can override by specifying higher priority.

Report will be sent in a json format to 
```html
.well-known/private-click-measurement/report-attribution/
```

Can use this info to measure ad campaigns without needing to track users.  

## Expand campaigns with app to web

```xml
<dict>
	<key> NSAdvertisingAttributionReportEndpoint</key>
	<string>https://example.com</string>
</dict>
```

We use this for the well-known path.  Exactly the same as the web case.

```swift
UIEventAttributionView()
```

When you display an ad, create one of these.  Verifies that a gesture has happened before forwarding attribution, so you know that reporting data is accurate.

When user taps the ad,

`UIEventAttribution(sourceIdentifier...)`

1.  Source identifier (ad campaign)
2.  destination URL (where converted)
3.  source description (content tapped)
4.  purchaser (buyer of the content)

```swift
let options = UIScene.OpenExternalURLOptions()
options.eventAttribution = ...
self.view.window?.windowScene?.open(adURL, options: options, completionHandler: nil)
```

For UIApplication-based lifecycle, use dictionary with `.eventAttribution:eventAttribution`.

## Fraud prevention
PCM uses cryptographic signatures to prevent fraud.

When app tap occurs, browser fetches store public key.  Report encrypted with public key.

When the browser removes the message from the envelope, it will have the signature.

Now source and destination site can validate that the click is trustworthy using store's pk.  

## Testing
Experimental Features => Private click Measuremenet Debug Mode

Also on iOS.  Enhanced inspector logging, reports sent every 10 seconds.



# Improve app to App Store campaigns

## SKAdNetwork overview
[[Build Trust Through Better Privacy]]
[[What's new with In-App Purchase]]

1.  Ad networks
2.  Publisher apps
3.  Advertised apps

`registerFor...`
`updateConversionValue`

## Reporting
SK combines information to a report and sends to ad network.

## Improvements

### Signing key changes
* Stronger signing key 256-bit
* Update apple public key
* See link to docs

### View-through attribution
`SKAdImpression`
startImpression
endImpression

SK introduces a new parameter called "Fidelity type"

view-through impressions have a fidelity type of 0.  We report highest fidelity.

### Multiple postback
& IP address protection.

Sending reports to a winner network and up to 5 runner-ups.

When a report is generated, the winner has the possiblity to... controlled values.  source app, conversion value.
Runner-ups do not get this info.

### Postback to developer
* Winning postback reported to developer.

```xml
<dict>
	<key>NSAdvertisingAttributionReportEndoing</key>
	<string>https://example.com</string>
</dict>
```

Same as PCM app to web rpoerting.  Post backs to `.well-known/skadnetwork/report-attribution`.

### Best practices
* Unique nonce
* Signature argument order.  Checkout docs to ensure arg order is correct.
* Source app ID (=0 for testing postbacks)
* Testing profile => can download from resources

# Wrap up
* Private Click Measurement
* SKAdNetwork updates
* Feedback

* https://developer.apple.com/documentation/storekit/skadnetwork
* https://developer.apple.com/app-store/user-privacy-and-data-use/
* https://webkit.org/blog/11529/introducing-private-click-measurement-pcm/
* https://developer.apple.com/safari/download/
* https://webkit.org
* https://developer.apple.com/bug-reporting/
* https://developer.apple.com/documentation/storekit

