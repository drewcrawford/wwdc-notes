#appclips 

# An app clip experience
* NFC: An NFC tag can encode an app clip URL.
* QR
* Maps
* Siri Nearby suggestions
* Safari -> banner
* When the user receives a link in a message, they get an "open" button for appclips
* App Clip codes.  "Later this year"


# Steps
* Configure web server and app clip for link handling
* Configure app clip experiences on App Store Connect
* Configure the Smart App Banner to open app clip (safari banner)

## Configure web server and app clip for link handling
* Update the `apple-app-site-association` file
* app clip: add associated domains entitlement
* Handle incoming `NSUserActivity`

### Update the file
```js
{
	"appclips": {
		"apps": [ "ABCD.example.my.app"]
	},
}
```

### Add entitlement
(not shown)
### Handle `NSUserActivity`
```swift
@main
struct AppClip: App {
	var body: some Scene {
		WindowGroup {
			ContentView()
			.onContinueUserActivity(NSUserActivityTypeBrowsingWeb) { userActivity in
			guard let incomingURL = userActivity.webpageURL,
			let components = NSURLComponents(url: incomingUrl, resolvingAgainstBaseURL: true) else {
				return
			}
			//direct to the linked content in your app clip
		}
	}
}
```

Keep in mind that once the user upgrades to your app, this will hit inside your app instead.  So make sure your app handles this.

Scene  UIKit version:

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
	func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
		guard userActivity.activityType == NSUserActivitytypeBrowsingWeb, let incomingURL = userActivity.webpageURL, let components=NSURLComponents(url: incomingURL, resolvingAgainstBaseURL: true) else {
			return
		}
		//direct to the linked content in your app clip
	}
}
```

[[What's new in universal links - 19]]
[[What's new in Universal Links]]

### Launching app clip from Xcode
Specify `_XCAppClipURL` environment variable

## Configure app clip experience
Metadata: title, 18 chars limit
subtitle: 43 chars limit
Image: 3kx2k
aspect ratio: 3:2
Format: png/jpg
Transparency: no

Go to app store connect.  After deliving a build containing both your app and app clip, it should appear in ASC.

### Advanced app clip experience
Multiple experiences, each with a URL

e.g. restaurant: one for ordering, and another for reserving a table.

URL mapping based on the most specific prefix match against registered experience URLs.

You don't need to register every possible URL.  However, your app must be able to handle being launched with the exact URL.  e.g., from siri nearby suggestions/maps

#### example: bike rental
`https://bikesrental.example/rent?bikeID=2`
This bikeshop does not need to register an experience for every url (bike).  CAn simply register for
`https://bikesrental.example/rent` for all bike URLs.

#### example: coffee shop
`https://brighteggcafe.example/store/campbell`
Each location offers similar experience.  `https://brighteggcafe.example/store/`  -> same card

But if you want a special card for one location, can register
`https://brighteggcafe.example/store/cupertino`.  

Basically, you can register
* a short URL that covers most cases
* A specific URL for specific cases

[[20/What's new in appstore connect]]
[[Design Great App Clips]]

## Configure the Smart App Banner to open app clip
```html
<meta name="apple-itunes-app" content="app-clip-bundle-id=com.example.fruta.Clip, app-id=12345789">
```

Note the `app-id` here shows an app banner on systems < iOS 14.

# Demo

# TestFlight
[[20/What's new in appstore connect]]

Can also add test invocation points for app clips.
Click on "Add App Clip Invocation" and set up title and URL.
