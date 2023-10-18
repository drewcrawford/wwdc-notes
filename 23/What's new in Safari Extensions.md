#safari 
Learn about the latest improvements to Safari extensions. We'll take you through new APIs, explore per-site permissions for Safari app extensions, and share how you can make sure your extensions work great in both Private Browsing and Profiles.

First, we have 2k+ safari extension on the app store.  Your extensions empower users to customize the browsing experience across macOS, iOS, etc.
* content blockers
* share extensions
* app extensions
* web extensions

safari 17 supports both manifest v2 and v3
web exetnsions are the best way to build extensions across platforms.
customize safari on iOS, iPadOS, macOS, now xrOS.

just like you'd expect, same capabilities, etc.  inject scripts, run bg content, display popovers.

[[Meet Safari for spatial computing]]

# New APIs
* content blockers

now support 'has' selectors.  In this rule exapmle,we're hiding ... ?

if you're looking to block/modify, check out 

## declarative net request
* block and modify
* power efficient
* Private and secure

* `declarativeNetRqeustWithHostAccess` permission.
* Required for `modifyHeaders` and `redirect` actions
* Per-site permissions required

use `setExtensionActionOptions` you can display badge text, for example.
count automatically updates
allows for easy monitoring of extension activity

## dynamic content scripts
`scripting.registerContentScripts` API
registered, updated, or removed programmatically

complements static content scripts, giving you greater flexibility in managing content scripts.

## temporary session storage
stored in memory
cl eared when safari quits

safari will take care of scaling your extension's icon, etc.



# Per-site permissions for Safari app extensions
Let's talk abuot safari app extensions for site permissons.

Per-site permissions work the same way for app extensions.

When an extension is first turned on, it doesn't have access to any site.  When it tries to access, we badge the toolbar item.

what access the extension will have, etc.  When granted permission, the toolbar will be tined to show the extension has access to the current page.

full user contro.
access can be granted or revoked at any time
toolbar items shown

# Profiles and Private Browsing
* access to private windows and tabs
* disallowed by default for extgensions that access page content
* allowed by default for content blockers

* separate data for differnet contexxt
* customized extensions
* sync across devices

when an extension is turned on in a profile, it's a new instance.  New uuid, background page, storage
per-site permissions are shared acros s profiles.  Only grant access 1nce
access to only the windows and tabs in that profile
profile identifier for messages to your extension's app

inspecting background content across profiles.  Develop->web extension background content.  Each thing lists its content per profile

[[23/What's new in Web Inspector|What's new in Web Inspector]]
[[Rediscover safari developer features]]

# Wrap up
safar is committed to standardize these extensions, etc.


adopt new APIs in safari, etc.

use fb assistant, etc.


[[What's new in Safari Web Extensions]]
[[Meet Safari Web Extensions on iOS]]



# Resources
* https://developer.apple.com/documentation/safariservices/safari_web_extensions/adding_a_web_development_tool_to_safari_web_inspector
* https://developer.apple.com/documentation/safariservices/safari_web_extensions/developing_a_safari_web_extension
* https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API
* https://developer.apple.com/documentation/safari-release-notes
* 