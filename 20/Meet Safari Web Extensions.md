#safari 

# Existing extension ecosystem
* content blockers
* share extensions
* safari app extensions

these are native technologies.  What about web technologies?

# Safari web extensions ( #macOS)

Web development model
compatible with other browsers

# Creating safari web extensions
packaged with native apps.  Up to you if the native app should play any role beyond this.
Distributed through the app store.
You'll need to download xcode 12

CLI tool

`xcrun safari-web-extension-converter [options] path/to/extension`

Run once
Add larger icons.  We recommend 512x512 and 1024x1024
Add files to xcode project

Note that by default, safari doesn't show extensions from ad-hoc signed apps.  Need to enable in developer menu.


# Considering privacy and permissions

## Demo: per-site extension permissions

Waring icon indicates app wants access on this site.
Choice is remembered across pages on that site.
Toolbar is highlighted indicating it can access data on this page.

## Optional permissions
Not critical to functionality.
add  a URL match pattern in the manifest for `optional_permissions` key.
In JS, call `browser.permissions.request` with the origin we want access to.


## Best practices
`activeTab` permission.  With this permission you're only granted to know things about a tab, when the user expresses intent to use your extension on that tab very quickly.

`optional_permissions`.  Not critical to the core functionality.

For some extensions you dont' want to require a user to take action.  You can state your intention to inject script on that page, and we will do a prompt.

# Debugging
Use `browser.?.getURL`

Debug->web extensionbackground page -> extension

Or find a webpage where it was injected and use web inspector.  Extension scripts will appear in a folder.

Need to switch to extension's world in the bottom right – extensions execute in an isolated world.

# Bugs
* User agent checks
* Hardcoded extension resource URLs.  Safari randomizes base url for osme reason
* Reliance on script injection time.  User may navigate first and then inject.
* Implementation differences.  


# communicating with your app
Native messaging
Extension can *only* communicate with the native app.

`browser.runtime.sendNativeMessage(...)` from background page
Need native messaging permission in manifest.

Other way
`SafariWebExtensionHandler.beginRequest`.

From app to extension, `SFSafariApplication.dispatchMessage`.  Make sure extension is turned on.  APIs for this.

To receive in the background page, you must have opened a port on background page.

Between app and extension, NSUserDefaults or XPCConnection.

## Demo
