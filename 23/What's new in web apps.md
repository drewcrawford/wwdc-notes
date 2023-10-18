Discover web apps for Mac — a powerful way to experience your website from the Dock. Learn how you can customize your web app to give people the best experience when they add your site. We'll also share how to take advantage of push notifications and badging for web apps for Mac and Home Screen web apps for iOS and iPadOS.

in 16.4, we added notifications to home-screen webapps.  We added API for iOS/iPadOS browsers to adopt at the home screen.

New in iOS/iPadOS 17, at the phone screen is now vailable in sfsafariviewcontroller. Can now add home screen webapps there.

In sonoma, we have webapps on mac.  Let me focus on websites that I use all the time, in a dedicated way that's separate from the rest of my browsing.  The way I can create a webapp is by adding  a website to the dock.

safari copies website cookies when added to the dock.

Simplified toolbar.
# Out of the box features
many features you would expect from a native app on macOS.
stage manager, cmd-tab, etc.

open from the dock or spotlight.
autofill credentials from iCloud keychain.
grant camera/microphone/location access

# Customizing your web app
you may want to customize your experience.
as a developer, you can control the initial behavior of the toolbar.  Default behavior has navigation controls.  Helpful for navigating around sites that don't have their own nav controls.  You may want to hide toolbar if you have your own nav controls or they aren't necessary.  

*standalone display mode*.

Websites that have been added to the homescreen in standalone display mode will become a homescreen web app.  Standalone app-like experience on IOS with separate cookies/storage from the browser.  All content is from the webpage, no UI.

If you want your site to use web push / badging on iOS, use the standalone display mode.

1.  Add webapp manifest `rel-"manifest" href="manifest.json"`
2. Add keys/values.  Name, `"display":"standaline"`.
	3. on macos, you hide the toolbar
	4. on iOS, you get a web app instead of open in browser
5. scope (outside links).  Default scope is host of webpage.  Can further refine the scope in the manifest to limit to a specific path on your site.
	6. `start_url` -> loaded when first opened
	7. `scope` -> separate into different webapps, etc.

Authentication state
* use cookies for authentication state
	* safari copies cookies to webapp when added
	* Local storage is not copied when webapp is created
* Ensure login flows stays within webapp
* OAuth implementation
	* heuristics to keep this in the webapp
	* please send us feedback
	* use window.open as a workaround.  Links here always open in webapp regardless of scope
	* we're working on standards for this
* Emailed login links
	* won't automatically sign users into the webapp
	* may want to provide a one-time code they can easily enter into the signin flow
[[Meet passkeys]]


# Notifications
Exciting addition to our existing standards-based webpush support.  

How you can integrate notifications, including badging/sounds.

Standards based web push
use web app icons
notification sound

* iOS and iPadOS sound on by default
* macOS sound off by default
* can override via `silent:true` in options, etc.

badging
webapps on amc support badging.  Since these are closely associated, when users allow a webapp to send notifications, that includes permissions for the webapp to use badging.  Badges can be updated when the webapp is updated or when push events are opened in the background.

[[Meet Web Push for Safari]]

Integrate with focus to get control over notifications.
`id` defines unique webapps across domain.  Multipel parts of your website that should be treated as distinct apps inside the same domain.  If you only have 1 webapp per domain, don't need to set the domain.  Fallback is the start url.

| name          | id     |
| ------------- | ------ |
| shop          | shop   |
| forums        | forums |
| forums - work (user creates extra app) | forums       |

related web APIs
* user activation api
* Fullscreen API
* screen orientation API

# Wrap up
* out of the box
* Web app manifest
* Web push and badging
[[Rediscover safari developer features]]
[[23/What's new in Web Inspector|What's new in Web Inspector]]
# Resources
* https://developer.mozilla.org/en-US/docs/Web/API/Push_API
* https://developer.apple.com/documentation/safari-release-notes
* https://developer.apple.com/documentation/usernotifications/sending_web_push_notifications_in_web_apps_safari_and_other_browsers
* 