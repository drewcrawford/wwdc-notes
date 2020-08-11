#wkwebview #webkit #safari 

# SFSafariViewController
In-app web browser without deep customization of the experience.

Reader, content blocker, autofill, etc.  
Easy to use but powerful, becaus eit is built on top of WKWebView.

When you need a higher degree of configurability, use WKWebView.

isolates in a separate process.  Strongly recommended over other alternatives.

WebView and UIWebView were our original APIs.  They were deprecated for a few years.  

**The WKWebView API is still growing**.

# New features
## Isolating your app from web content
* `WKPreferences.javaScriptEnabled`.  Now deprecated, now `WKWebPagePreferences.allowsContentJavaScript` only disables JS from remote content.  Your app's JS will continue working.  You do this in `decidePolicyFor` event.

For years, you've been able to use the web inspector in your simulator / app, etc.

Your app can accidentally conflict with the web content.  Malicious content might try to conflict with you on purpose.  Need an isolated place for our JS to run.  `WKContentWorld`.

`WKScriptMessageHandler` inject into a specific world.

## Communicating with javascript
`callAsyncJavascript`.  

```swift
let styleJavaScript = """
	var element = document.getelementById(elementIdToStylize);
	if !(element)
		return false;
	for (const theStyle in stylesToApply)
		element.style.theStyle = stylesToApply[theStyle];
	return true;
"""

webView.callAsyncJavaScript(
	styleJavaScript,
	arguments: [
		"elementIDToStylize":"postContainer",
		"stylesToApply":["margin:0,"padding-left": "5px"]
	],
	in: .defaultClient,
	completion: { _ in /* check return value if desired */ }
)
```

Serialization and deserialization happens automatically.  Evidently with Cocoa types.

If your JS returns a promise, then your completion handler will wait for the promise and will be called with the result.

`postMessage` used to return `undefined`.  Now it reutnrs a promise.
Second, a new handler for `WKScriptMessageHandlerWithReply` that call will be resolved.

## More flexible rendering
`WKWebView.pageZoom`.  Drives cmd+ and cmd- full-page zoom.  This avoids conflicts with the page if you want to do it natively.

`WKWebView.mediaType`.  Can now make up your own CSS media query selectors, so you customize the native situation.

`WKWebView.findString`.

`webView.createPDF`.  Can now share the whole page, not just what's visible.

`createWebArchiveData`.  Can now create web archives.

`printOperationWithPrintInfo`.
## Respecting privacy
ITP.  ITP is now enabled by default on all WKWebView apps.

three ways web is used in an app
* general-purpose browsing
* only one or a few sites.  web content is part of the app.
* In-app browser.  e.g., a social media newsfeed, that starts from a known source and browses anywhere

## app-bound domains
Specify which domains are the core part of the implementation.  Can implement principle of least privilege, 

`WKAppBoundDomains` array in info.plist
Loading other domains still works, but "deep interaction with other domains is disabled"
Can also disable interaction with all domains, by adding `WKAppBoundDomains` as an empty array.  If you load arbitrary content but don't interact with it, this is a best practice.




## Working with web content (native?)


