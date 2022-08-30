# What's new in Safari Web extensions
I'd like to take a moment to thank all of you for submitting for iOS, macOS extensions to the app store.  Moving forward, our goal is to continue to implement features and APIs to deliver even better experiences.

# manifest verison 3
Next iteration of web extension platform.  Performane and security improvements.  Safari supports manifest 2 and 3.  (15.4+).

For those who haven't, we will continue to support v2 in safari.

## Service workers can replace background pages
* Familiar web technology
* Compatible with other browsers
* Substitute with non-persistent background pages


* ## Executing script on webpages
* new ways to execute code
* more frame options for injections
* main world execution

Let's take a look at how code for scripting different from tab.

```js
// Manifest version 2
browser.tabs.executeScript(1, {
  frameId: 1,
  code: "document.body.style.background = 'blue';"
});
```

can only inject code that's contained in a string.  But now...

```js
// Manifest version 3
function changeBackgroundColor(color) {
  document.body.style.background = color;
};

browser.scripting.executeScript({
  target: { tabId: 1, frameIds: [ 1 ] },
  func: changeBackgroundColor,
  args: [ "blue" ]
});
```

can contain arguments, etc.  You're not confined to writing code in a script.  New property called `target`.  Specifies where script should run.  Specify the id of tab you want to execute.  API will return error if tab id is not specified.

v2 Only specify 1 id.  But with scripting, you can specify multiple id.

What about multiple files?  can now say `files:[]` array.  CSS, removeCSS, etc.

```js
// Manifest version 3
browser.scripting.executeScript({
  target: { tabId: 1 },
  files: [ "file.js", "file2.js" ]
});
```

```js
// Add styling
browser.scripting.insertCSS({
  target: { tabId: 1, frameIds: [ 1, 2, 3 ] },
  files: [ "file.css", "file2.css" ]
});
```

```js
// Remove styling
browser.scripting.removeCSS({
  target: { tabId: 1, frameIds: [ 1, 2, 3 ] },
  files: [ "file.css", "file2.css" ]
});
```




These APIs are available in v2,v3.  However, tab.executeScript is not available in version 3.  In addition to new scripting API, there's a slight modification to other APIs as well.  One modification is web accessible resources.

## web accessible resources
In v2, you provide an array of files.  But this gives any website access to the files.

In v3, you control which resources are avilable on any given site.  ex
```js
// Manifest version 3
"web_accessible_resources": [
    {
      "resources": [ "pie.png" ],
      "matches": [ "*://*.apple.com/*" ]
    },
    {
      "resources": [ "cookie.png" ],
      "matches": [ "*://*.webkit.org/*" ]
    }
]
```

Now let's take a look at modification to browser action.  In v2, we specified page actions distinctly.  Now we consolidate to use 1 api, "action".

```js
// Manifest version 3
"action": {
  "default_icon": {
    "16": "Images/icon16.png"
  },
  "default_title": "defaultTitle"
}
```

## security policy
in v2, string

```js
// Manifest version 2

"content_security_policy" : "script-src 'unsafe-eval' https://*apple.com 'self'"
```
in v3, object.  Important to note that remote sources for scripts are not allowed in v3.
```js
// Manifest version 3

"content_security_policy" : { "extension_pages" : "script-src 'unsafe-eval' 'self'" }
```
`browser.extension.getURL()` is deprecated.  Instead use version in get runtime

## Updating process
This extension replaces all fish with emoji.
1.  change manifest_version to 3.
2. Update background page with a service worker
3. `browser_action` => `action`
4. Build extension
5. enable in safari

Use the extension to replace every instance. But it didn't work.

See errors in web inspector.

`browser.tabs.executeScript` is not a function.  No longer available in v3.

Use `browser.scripting`.  Add target property to specify where injected.  

Get object contianing info of the current tab.  `browser.tabs.getCurrent()`.  Use that object to retrieve the tab ID.

Add file containing script to run.  Add `scripting` permission.

Now works in v3.  

> that's how simple it is

Scripting and service workers are also available in v2.

# Updated APIs

## Declarative Net Request
content-blocking API to privately block requests with rulesets.

Intercept and modify gets delegated to safari.  You specify the rules.

```js
// manifest.json

"permissions": [ "declarativeNetRequest" ],

"declarative_net_request": {
  "rule_resources": [
    {
      "id": "my_ruleset",
      "enabled": true,
      "path": "rules.json"
    }
  ]
}
```

previously, 10 rulesets.  Now up to 50.  Only 10 can be enabled at once.

[[Explore Safari Web Extension improvements]]

Previously, you can only declare rulesets in the manifest.  But now we have 2 APIs to update dynamically.

* updateSessionRules => add or remove rules.  These rules will not persist across browser sessions or extension updates
* updateDynamicRules => update blocking rules without updating entire extension.
```js
// Rules that won't persist

browser.declarativeNetRequest.updateSessionRules({ addRules: [ rule ] });

// Rules that will persist

browser.declarativeNetRequest.updateDynamicRules({ addRules: [ rule ] });
```

## Demo
In extension's manifest, we add the `declarativeNetRequest` permission.

Use key to add a ruleset in our manifest.  Rules Located in `rules.json`.

Suppose want to update all pages except webkit.org pages.  We can update dynamically.  

# `externally_connectable`
Declare URL match patterns in the manifest.  Determines which pages can communicate with the extension.
Must use the `browser` namespace
User must grant permission.
```js
// In the webpage
let extensionID = "com.apple.Sea-Creator.Extension (GJT7Q2TVD9)";

browser.runtime.sendMessage(extensionID, { greeting: "Hello!" },
 function(response) {
    console.log("Received response from the background page:");
    console.log(response.farewell);
});
```

Can find your team identifier on developer.apple.com in the membership tab in your account settings.

Now, let's take a look at the code your extension has to receive messages.


```js
// In the background page
browser.runtime.onMessageExternal.addListener(function(message, sender, sendResponse) {
    console.log("Received message from the sender:");
    console.log(message.greeting);
    sendResponse({ farewell: "Goodbye!" });
});
```
Determining the correcct identifier
* extensions have browser specific identifiers
* `browser.runtime.sendMessage()`
* `Promise.all`

Here we have a function, sends messages to the extension.  If you have multiple IDs and want to determine the correct one, can use `Promise.all` using the `determineExtensionID` function.
```js
// Determining the correct identifier

function determineExtensionID(extensionID) {
  return new Promise((resolve) => {
    try {
      browser.runtime.sendMessage(extensionID, { action: 'determineID' }, function(response) {
        if (response)
          resolve({ extensionID: extensionID, isInstalled: true, response: response });
        else 
          resolve({ extensionID: extensionID, isInstalled: false });
      });
    }
  });
};
```


```js
Promise.all([ determineExtensionID(extensionID_1), determineExtensionID(extensionID_2)])
.then ((extensions) => {
    var extensionObject = extensions.find(extension => {
		if (extension && extension.isInstalled)
			return true;
	});

	const extensionIDToUse = extensionObject.extensionID;
});
```

Reply to say we're installed
```js
// background.js

browser.runtime.onMessageExternal.addListener(function(message, sender, sendResponse) {
  if (message.action == "determineID") {
    sendResponse({ "Installed" });
  }
});
```

## Unlimited storage
* Actually unlimited!
* No longer have a 10mb quota.
* Use as much data s you see fit.
* Users can clear data.

```js
// manifest.json

"permissions": [ "storage", "unlimitedStorage" ]
```

Those were all the updated APIs.  
# Syncing extensions
Get extension on all devices.
* enabled state synced across all devices
* Easier downloads

In extension settings, they will be given the option to download extension.  Once download, it's automatically enabled on device improving their experience.

* List extension on all platforms
* Recommended: adopt universal purchase
	* users to enjoy extension across all platforms by only purchasing once.
* Not recommended: manually link bundle identifiers in xcode
## Associate iOS extensions
macOS app key: `SFSafariCorrespondingIOSAppBundleIdentifier`
macOS app extension key: `SFSafariCorrespondingIOSExtensionBundleIdentifier`

## Associate macOS extensions
iOS app key: `SFSafariCorrespondingMacOSAppBundleIdentifier`
iOS extension key: `SFSafariCorrespondingMacOSExtensionBundleIdentifier`


## Demo

## Recap
* either setup universal purchase
* link bundle identifiers in xcode


# Wrapup
* manifest v3
* apis
* syncing extensions

* download sample project
* Provide feedback
* Join WebExtensions Community Group
[[Create Web Inspector Extensions]]


* https://developer.apple.com/documentation/safariservices/safari_web_extensions/modernizing_safari_web_extensions
* https://developer.apple.com/documentation/safariservices/safari_web_extensions/messaging_a_web_extension_s_native_app
* https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API
* https://developer.apple.com/documentation/safariservices/safari_web_extensions
