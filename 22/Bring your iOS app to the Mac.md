
Whether you ship your app on m1 max, thining about going further with mac catalyst, or already ship a catalyst app you want to improve, I'me xcited to share new APIs/techniques.

Show off some amazing work done by developers taht showcases what's possible with mac catalyst.

Craft.
Darkroom.
Night sky.
Asphault 9 legends.

# iOS apps on M1
native iOS apps on macs with m1.  Your apps are already available on m1 on the mac app store.  We have a couple of new additions that can improve your experience on mac.

New launch options.
`UISupportsTrueScreenSizeOnMac` => indicates that your app is prepared for wide variety of display configurations it may encounter.  Your app gets the true screen size and pixel density rather than a compatible iPad size.
`UILaunchToFullScreenByDefaultOnMac`

These ignored on iOS and old mac.  Safe to add to any app that would benefit.

Provide an immersive experience immediately.  Pulls you itno its beautiful world of exploration.

Touch alternatives.  Automatically convert keyboard, mouse, trackpad input into iOS multitouch gestures and device motion expected by your app.  We've added built in support for the most popular games on the app store.  When launched, they show a tutorial explaining how touch controls translate to keyboard/mouse and trackpad.

Arrow keys - swipe from center of the window.  Spacebar can perform a tap.  Etc.

`com.apple.uikit.inputalternatives.plist`  This is a new plist file.  `defaultEnablement="enabled"` tells the system to make it on immediately.

`requireOnboarding`.  An array ocntianing a list of which controls you've decided work best for your app.

| touch control    | mac alternative |
| ---------------- | --------------- |
| tap              | spacebar        |
| tilt             | wasd            |
| scroll drag      | scroll gesture  |
| arrow swipe      | arrow keys      |
| Trackpad capture | hold option + Trackpad                |
Note that when enabled, allt hese controls will be active.  But you should still decide what makes the most sense for your app, only add controls you want *highlighted* to the plist.

In your app's settings, people can switch between displaying preferred controls from plist, and all controls.  

We do recommend keyboard support though.
[[Support Hardware Keyboards in your App]]
[[Handle trackpad and mouse input]]


# Build for Mac catalyst
By adding a mac catalyst destination, your app will automatiaclly be converted to a full catalyst app.  Capable of running on every mac, and letting you customize further with mac catalyst apis.

Optimize itnerface for mac, which gives you antive appkit-style controls and ensure you render at native scale.

Let's use the markdown demo app.

side by side.  UInavigationBar => NSToolbar.
Text size adjusts as well.  iPad idiom at iPad size, and then scaled to 77% of original size.
Mac idiom => native mac font rendering, which happens at pixel-eprfect scale.  Ideal for our app, since it ensures the text always looks crisp.
# What's New
[[Meet desktop-class iPad]]
[[Build a desktop-class iPad app]]

New APIs translate into native mac representations.

Controls and navigation move from UInavigationBar into NSToolbar.  If you don't arleady create a toolbar, we give you one automatically. 
If you've already managed your own NSToolbar in catalyst, we stay out of your way.

Center item controls become NSToolbar items.  For document-based apps, your window title shows the document name.  File proxy icon can appear as well.

Back button and other navigation controls are brought into the toolbar.  Additionall,y you get new document-centric menu items in the file menu.  Duplicate, move, rename, export as, etc.

Implement buildMenu emthod on app delegate to control your app menus.  New `.document` menu items.  

## Search bar
Automatically pulled into NSToolbar as well.  First showing as a search button, athat expands into the bar on click.


# New API
In addition to those, we've added several new catalyst-*specific* API to improve multiwindow and toolbar behaviors.

 * adopted mac idiom
 * pointer customization to indicate resize on hover
 * printing support

Don't feel like you need to add every feature.  Think about the type of app that you create and which features work best.  Check out mac/mac catalyst HIG, and look for inspiration in other apps you use.

* window geometry
* window controls
* NSToolbarItem hosting
* Popovers from toolbar

I'll use these new APIs to improve our appe ven further on mac, starting with windows.

## Window API
* Window geometry
* window controls
* Disable fullscreen

Show an auxiliary panel.  Good practice to always start with the curernt frame from `.effectiveGeometry`.  This is initialized to CGRectNull, whose values system knows to ignore.  We modify the size and give the scene a new frame, my giving it a `UIWindowScene.MacGeometryPreferences`.  Then we call `requestGeometryUpdate`.  Because this is a request, system may reject the new geometry.

At any time in a scene's life, you can check its frame with  ?  Modify it, etc.

### Notes
* coordinates always 1:1 with AppKit
	* Even if you've scaled to match iPad.
* Origin in top left of main screen (main display).  if you have multiple displays, the main display is the one that shows the menub ar in system display settings.
* With new mac catalyst API, you can take control over the state of each control.

* closable
* isMiniaturizable
disables red/yellow window buttons.
`.allowsFullScreen` can both resize and fullscreen.  You can disable fullscreen.  Or disable resize with min/max size to the same size.

By doing both, green button also becomes disabled.

Check whether currently full screen with `.isFullScreen` property

## Toolbar
UIViews can be added as items to the NSToolbar.  I designed a custom UIView that shows the current word count.  When clicked, it presents a popover with additional details, like paragraph, section counts, etc.

Custom view property on UIBarButtonItem is automatically wrapped and added to the toolbar.  But if you manage your NSToolbar independently, can also use NSUIViewToolbarItem.

  * Automatic conversion clones view
  * manual NSToolbar needs new UIView instance
	  * can't re-use instance


Opt out of navigation bar translation by using `.preferredBehavioralStyle = .mac`.  Default is `.automatic`.  By setting it to `.pad`, you no longer get translated.

With these options, you can add a new layer of customization to your app's toolbar.

# Next steps
* run existing iOS apps on M1 Macs
* Build your app with mac catalyst
* Adopt Mac Catalyst API


* https://developer.apple.com/forums/tags/wwdc2022-10076
* https://developer.apple.com/forums/create/question?&tag1=155&tag2=437030
* https://developer.apple.com/documentation/uikit/app_and_environment/supporting_desktop-class_features_in_your_ipad_app
* https://developer.apple.com/design/human-interface-guidelines/macos/overview/themes/
* https://developer.apple.com/design/human-interface-guidelines/ios/overview/mac-catalyst/
