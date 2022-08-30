#wkwebview 

Added a lot of new web technologies.  Before we start, let's maek sure you are using the right technology for your applications.

* SFSafariViewController
* UIWebView => migrate to WKWebView!  **will be removed in a future release**
* WKWebView is the API to use to write applications that interact with web content.  CSS-based UI, writing some of your app in JS.  Your own web content using app-bound domains.  Developing your own browser.

# Web content interaction
3 new ways.
* fullscreen support
* CSS viewport units
* Find interactions
## fullscreen
JS can make videos/canvas elements fullscreen.  Now can do it in your app as well.

```swift
webView.configuration.preferences.isElementFullscreenEnabled = true

webView.loadHTMLString("""
<script>
    button.addEventListener('click', () => {
        canvas.webkitRequestFullscreen()
    }, false);
</script>
…
""", baseURL:nil)

let observation = webView.observe(\.fullscreenState, options: [.new]) { object, change in
    print("fullscreenState: \(object.fullscreenState)")
}
```

## CSS viewport units
svh, lvh, dvh, many more!  Allow web developers to modify layout based on smallest, largest, dynamic viewport sizes.  

```swift
let minimum = UIEdgeInsets(top: 0, left: 0, bottom: 30, right: 0)
let maximum = UIEdgeInsets(top: 0, left: 0, bottom: 200, right: 0)
webView.setMinimumViewportInset(minimum, maximumViewportInset: maximum)
```

if you change the viewport, inform webkit upfront of maximum and minimum edge inset to allow it to layout correctly.

## find interactions
```swift
webView.findInteractionEnabled = true

if let interaction = webView.findInteraction {
  interaction.presentFindNavigator(showingReplace:false)
}
```

allows command-f, etc.

# Content blocking
New capability to WKContentRuleList.  
Now you can write a rule to match only certain frames.
```swift
let json = """
[{
    "action":{"type":"block"},
    "trigger":{
        "resource-type":["image"],
        "url-filter":".*",
        "if-frame-url":["https?://([^/]*\\\\.)wikipedia.org/"]
    }
}]
"""

WKContentRuleListStore.default().compileContentRuleList(forIdentifier: "example_blocker",
    encodedContentRuleList: json) { list, error in
    guard let list = list else { return }
    let configuration = WKWebViewConfiguration()
    configuration.userContentController.add(list)
}
```

This rule now only applies to requests from frames that match the regex.  

[[What's new in Safari Web Extensions]]  New possibilities for...

# Encrypted media
IF you have content, you can now use in apps on iPadOS.  
# Remote web inspector
Remote web extension will just work.  No need to add or change any code.  Settings=>Safari=>Advanced=>Web inspector.

Remote web inspector has many tools, etc.  Explore the dom, run/debug JS execution, view timelines, and more.  If you have a website, you can now inspect and debug it yourself in third-party browsers on iOS.

# Wrap up
* Try new APIs
* Use best technology for your app
* Give us feedback

[[What's new in Safari Web Extensions]]
[[What's new in Safari and WebKit]]










* https://developer.apple.com/forums/tags/wwdc2022-10049
* https://developer.apple.com/forums/create/question?&tag1=261&tag2=512030
* https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller
* https://webkit.org
* https://developer.apple.com/documentation/webkit
