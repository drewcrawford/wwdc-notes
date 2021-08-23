#webkit 

Powerful browser extension model.  Standard web technologies, HTML, CSS, and JS.  Write extension once and deploy across browsers.

Now, coming to iOS 15.  

Shows fun facts about sea creatures and makes learning more about sea life fun by replacing names of sea creatures with emoji.

Turn on from a dropdown in the right corner of address bar?

Swapped in all these fish emoji.  Permission to work across website as a whole.  Navigate to other pages, still working for me.  Don't need to open the extension again.

# Creating extensions
For safari, web extensions are parts of apps.  When you want to install a web extension, you install an app.

Like any other types of iOS apps, they're in the appstore.  And xcode builds them.

1.  Create a new safari web extension?
2.  Convert an existing extension?
3.  Add iOS support to a macOS web extension?

Start building web extension using app template.  When you use this template, you get an extension that comes complete with resources.  Use this as a starting point by customizing what's already there or adding/removing pieces.

```
xcrun safari-web-extension-converter [options] <path to extension>
```

xc13 adds supports for iOS.

Uses original resources from provided path
Add iOS support to a macos safari web extension.

Use `--rebuild-project` to add iOS support.  Converter will add an iOS-compatible version of the extension and containing app.

Persistent background page isn't supported on iOS.  

Folders for both iOS and macOS app in xcode.  

1.  Manifest => json file that describes structure of the extension.  Seems a bit like info plist.
2.  background.js.  Browser runs this in the background and allows extension to listen for various events.  
3.  content.js.  Browser runs this script on webpages that the user visits.  Extension can have any number of these, and the manifest specifies the mapping.


# Debugging techniques
How to dig deeper to find out why things go wrong?

Extensions on iOS must have a non-persistent background page.  

```json
"background": {
	...
	"persistent":false,
}
```

Use web inspector take a closer look at page rendering.

1.  Open up mac safari
2.  Develop menu enabled
3.  Develop=>simulator=>choose a page to inspect

By default, safari on iPhone renders a page at desktop size and scales down.  But I don't want this behavior here as it makes text too small.  So I'll use `viewport` to tell it not to scale content.

```
<meta name="viewport"...
```

Set max width for content rather than demanding an exact width.  `width` => `max-width`.    Now content's width is more appropriate for iPhone.

Save from safari and overwrite css for edited resources.  But for edits I made to DOM, need to copy the tag I edited.

* View manifest errors in safari's extension settings
* Run app to update the web extension
* Use web inspector to inspect web content

[[Discover Web inspector improvements]]

# Best practices
## BAckground page
Runs background script to handle events
This has a performance cost.  

Can be made non-persistent
	Loaded when needed
	Unloaded when idle

**Background pages must be non-persistent on iOS**

Opt-in existing web extensions in `manifest.json`

[[Explore Safari Web Extension improvements]]

## Responsive design
Test layout on iPhone, iPad.  use responsive design to accommodate various screen sizes.  Difference in size isn't the only consideration when it comes to making web content look great.

If your extension has full-page content, you may find it being covered by safari's tab bar or the device's home indicator.  e.g. unsafe area vs safe area.

By using CSS environment variables, you can calculate safe area insets to position within the safe area.

By using similar CSS and specifying viewport-fit, you can give your web content an edge-to-edge design that still keeps important content within the safe area.

[[Design for Safari 15]]

If your extension has a popup page, you may be used to a popover.  On iPhone, safari displays this as a sheet.  Sheet is full width and may be taller than your content expects.

Consider if you should make similar changes to your extension.  Safari will use a similar presentation in landscape as well.  

Dynamic type.  Be sure to test your extension's interface under smaller/larger text sizes to look OK with any size.  To help web content make the most of dynamic type, there's a variety of WK system fonts like `-apple-system-headline` etc.  Adopt these fonts in your extension so that its text remains comfortably readible.

* Design for safari's UI on iOS
* Test full-page web content on iPhone
* Test popup web page on iPhone
* Test with Dynamic Type sizes

[[Design for Safari 15]]

## Pointer events
`element.onmousedown` vs `element.onpointerdown`.  Be aware that mousedown isn't sent for tap.

Adopt pointer events API.  The same with mouse input, but also reports touches and apple pencil input.


## Window APIs
On desktop browsers, users may have multiple windows open and your web extension can use `browser.windows`.  Same is true on iPad where you can also open multiple windows of safari.  Might be fullscreen, or might be splitview.

`incognito` `focused` properties on windows.

One safari scene is represented by two windows.  But if I open a scene in splitview, I get 4 windows.  Windows.oncreate fires twice.  Keep this model in mind.

`windows.create()`, `remove()` and `update()` are not supported.  Window placement is controlled by the user.
`windows.onRemoved` won't be fired for closing safari
These apply only to `browser.windows` not `browser.tabs`.  You still can add remove, update tabs.
## Feature detection
### unavailable apis
* windows APIs
* contextMenus
* webRequest

## feature detection
`if (browser.contextMenus) { ... }`

Use this pattern for new APIs that are added in the future.


# Privacy considerations
Opt-in model where extensions are only given access to websites when the user consents.  I got a prompt "Allow for One Day" "Always Allow"

Safari tells me in address bar that I have an extension running.  If I navigate to any website that I haven't allowed the website to access, that indicator disappears.

## Permissions
* Users opt into your extension *per website*
* Safari asks for user consent
* Permission is required for any privacy-sensitive API
	* Tab URLs and titles
	* Cookies
	* Script and stylesheet injection

If you call these without permission, safari will wait to call your completion handlers and will show a banner at the top of the screen.  This lets the user know your extension wants some more access and can review set of websites.  Allow/reject access per website.  

Avoid requesting more permissions this way than you reall need.  For some types of extensions, this way of thinkign about permissions may be more than you need.  e.g. share or annotate particular website, just once to do one thing.  For this case, we have `activeTab` permission.

### activeTab
* Granted when the user invokes your extension implciitly
* Limited to current website in current tab
* Does not require a prompt
* Add permission to `manifest.json`

## Privacy wrapup
* Users control which websites extension can access
* Consider using the `activeTab` permission
* Same across macOS, iOS, and iPadOS

[[Meet Safari Web Extensions]]

# Wrap up
* Download sample code
* Convert your extension for iOS
* File feedback
* Apple developer forums

[[Explore Safari Web Extension improvements]]
[[Design for Safari 15]]

* https://developer.apple.com/safari/download/
* https://developer.apple.com/documentation/safari-release-notes
* https://developer.apple.com/bug-reporting/

