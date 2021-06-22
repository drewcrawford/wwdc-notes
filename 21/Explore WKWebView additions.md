#wkwebview 

When a user clicks a URL, they're expecting a webpage to load.  Rendering the content and running JS in that content.

## Safari
Used to show all web content on iOS and much of macOS.

## SFSafariViewController
Don't need much customization, you probably just need this.

Tiny safari inside your app.  Using one of these views is incredibly simple, but there is a tradeoff.  Ltitle you can do to interact with the web content.

Users have always been able to run app extensions through share sheet.  But, for special functions built only for your app, it's been cumbersome.

We've added a new API to bring one of your app extensions to a customized buttons.  Can even set an image for that button best representing the extension.

Run JS on the page, etc.

But this is still a very limited amount of interaction

## WKWebView

Loading, manipulating, interacting with content.  So if you need complex interactions, this is for you.

1.  Avoid JS
2.  Access to functionality previously only available in Safari.

## JS
Injecting JS is complex.  May have unintended side effects or be difficult to manage.  Best to avoid taht headache if possible.

Features that are incompatible with injected JS.

App-bound domains.  Specify which domains you allow deep interaction with.  Helps increase security/privacy of your app.

To get this benefit, you cannot inject JS.  Doing so disables the feature.

Don't have applepay if you inject JS.

Therefore, we've added new APIs to interact without JS.

### Access theme color
Theme color.

[[Discover WKWebView enhancements]]

`webview.underpageBackgroundColor`
some other theme color as well?

Observable.

### Manage text interaction

Disable text interaction when playing videos.

`preferences.textInteractionEnabled = false`

turns off all text interactions.  

### Control media playback

* Previously only through JS
* Hd to find the specific element
* Simple API on WKWebView

`await webView.pauseAllMediaPlayback()
closeAllMediaPresentations()
requestMediaPlaybackSTate
setAllMediaPlaybackSuspended(true)
`

Post videos instead of still images.  App has videos now too.

## Demos

This JS was my only option to try to pause before.  It's been problematic.

JS changes for the page.  First I adopted `.pauseAllMediaPlayback`.  Equivalent to calling JS function `pause` for every element on the page.

But when I refresh the page. New videos have never been paused.

`setAllMediaPlaybackSuspended(true)` => applies to future videos.  Property of the webview itself, not of any content.

You now have the flexiblity to add a better media experience.  

## Browser-level APIs

* Disable HTTPS upgrade
* Control media capture, `getUserMedia`
* Manage downloads

### HTTPS

Always searching for new ways to make security/privacy easier for you and your users.

Safer, more secure way to browse the web.  

If you do need to turn this off, `upgradeKnownHostsToHTTPS`.  Hopefully you don't need this.  Probably don't do this in production

### Media capture

* WKWebViews started support for `getUserMedia` for web RTC functions
* Origin of request can be your app
* To remain as the URL, load without the custom scheme handler
* Decide when to re-prompt the user

Once you've obtained user permissions via entitlements and prompts, you can decide if the prompts can be shown.  `decideMediaCapturePermissionFor origin: initiatedByFrame: type:`.  

To skip the prompt, delegate will allow you to do that.

WebRTC functionality.  

We know that requests from our server are ones that the user will already give permission to, so skip the prompt with delegate.

Get/set state without JS.  `webView.setCAmeraCaptureState` `setMicrophoneCaptureState`.

#### Demos


## Downloads

API that lets you allow/manage downloads from webview.

* Web content via JS
* Server via HTTP
* App via API

JS > calls navigation delegate with `shouldPerformDownload` decision.

Server initiates a download via HTTP.  In that case, navigation response will have a "attachment".  Return `.download` to allow this.

Finally, app can decide to download something via current page using NSURLRequest.

* Get a WKDownload object
* Set a WKDownloadDelegate
* Write the file

If it fails, data to resume the download will be handed to you via delegate method.

With this API, can download tis to a file.

New options for rich web experiences. 
# Wrap up
* Theme color
* Text interaction
* media playback
* HTTPs override
* Media capture
* Downloads

[[Develop advanced web content]]
[[Discover Web inspector improvements]]

webkit.org has multiple ways to reach out.  Slack workspaces, mailing list.


