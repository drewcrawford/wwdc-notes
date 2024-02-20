Explore the latest updates to App Clips. We'll show you how to build App Clips more easily using default App Clip links. Learn how you can take advantage of the increased App Clip size limit to build richer and more engaging experiences, and find out how you can launch App Clips directly from your app.

Users can discover app clips throughout the system.  Messages, maps, safari, etc.  Scan codes, etc.  WE've built 3 new improvements to app clips experience.

# New size limit

Variety of other usescases.  User may be on a fast home network, etc.

To enable an even broader set of experience.  New 50MB size limit for digital invocations.

* allows for richer digital invocations
* physical invocations (NFC tags or app clip codes) is still 15MB.
* Enhance your App Clip functionality
* Provide more immersive experience
* iOS 15 or earlier, 10MB limitation applies.


# Default app clip links
Many app clips only require a single experience.  Default experience, the most general usecase, encompass the core functionality of your app.
Invoked using universal link.
https://naturelab.example.com/


[[Configure and link your app clips]]
Default app clip links are a new way to invoke app clip experience.  Generated URLs that are created for you when publishing an app clip in asc.

Default experience - invoked without any extra steps.  Supported started in ios 16.4.

https://appclip.apple.com/id?p=<bundle_id>&key=value

### Parsing URL parameters as components - 3:53
```swift
ContentView(parameters: $parameters)
    .onContinueUserActivity(NSUserActivityTypeBrowsingWeb, perform: { userActivity in
        guard let inputURL = userActivity.webpageURL else {
            return
        }

        let components = NSURLComponents(url: inputURL, resolvingAgainstBaseURL: true)
        guard let parameters = components?.queryItems else {
            return
        }

        self.parameters = parameters
    }
```

### Providing metadata to an LPLinkView - 4:39
```swift
let provider = LPMetadataProvider()

provider.startFetchingMetadata(for: url) { (metadata, error) in
    guard let metadata = metadata else {
        return
    }

    DispatchQueue.main.async {
        lpView.metadata = metadata
    }
}
```

# invoke from your app

Users love the ability to launch your app clips from various parts of the system.  Bring that functionality directly to your apps as well.

Link presentation API - tappable rich preview to allow it to be invoked.  Onc eyou've retrieved your metadata... render a preview.

Can invoke app clip link directly.

This linking behavior works without safari, etc.





### Launching App Clips from a SwiftUI app - 5:00
```swift
var body: some View {
    let appClipURL = URL(
        string: "https://appclip.apple.com/id?p=com.example.naturelab.backyardbirds.Clip"
    )!

    Link("Backyard Birds", destination: appClipURL)
}
```

### Launching App Clips with UIApplication - 5:11
```swift
func launchAppClip() {
    let appClipURL = URL(
        string: "https://appclip.apple.com/id?p=com.example.naturelab.backyardbirds.Clip"
    )!

    UIApplication.shared.open(appClipURL)
}
```

# wrap up
* build richer app clip experiences
* set up easily with default App Clip links
* Launch app clips from your app
[[Configure and link your app clips]]



# Resources
* https://developer.apple.com/documentation/app_clips
* https://developer.apple.com/design/human-interface-guidelines/app-clips
* 