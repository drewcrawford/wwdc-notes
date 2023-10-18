Similaries are more than skin-deep.  Same engine, etc.

All websites work out of the box, etc.

I like using this iwth the mac virtual display, etc.

bringing your favorite web videos into fullscreen will bring them into focus.

* CSS viewport units
* media and container queries
* Vector graphics with SVG
* `devicePixelRatio` for bitmpa assets
* responsive images

# natural interactions
pointerdown 
pointermove, pointermove, etc.
pointerup  - on release

direct gestures -> touch the page.  Pointerdown is dispatched based on hand position.  move will have motion, etc.  pointerup -> dispatched when finger stops intersecting with the window.

safari's scroll etc work as expected.

For media queries, primary input model is similar to a touchscreen.  pointer is `coarse` and there is no hover support.  But a trackpad/keyboard could be connected via bluetooth.

this device comes with a new way to provide visual feedback.

interactive regions (highlight support)
* buttons, links, menus
* eleemetns with the equivalen tARIA roles
* input fields and form eelements
* elements with CSS `cursor: pointer`.

cursor property is inherited.  So the icon and label will get their own region and highlight separately.  so I can set `pointer-events:none` to avoid individually highlighting.

can also ahep the highlight bey using CSS `border-radius`.  Important to match the visual style of each element.

use simulator to debug highlight regions.  Recall that safari uses the cursor CSS property to determine if an element should be interactive.  

If your interacting element has a border radius, we take that into account.  safari has a small radius by default.


# Platform optimizations
when you request to go fullscreen, focus on a single element.  Page fades away, window is no longer fullsize, etc.

window reported bakc to JS in screen dimensions.  Websites expect fs to match.

remember that fullscreen windows can be resized on this platform, adn they can get larger than the reported screen dimension.

* high framerate for all types of animation
* passive event listeners for scroll
* time deltas for `requestAnimationFrame`
* web inspector timeline

[[Web inspector walkthrough]] - tech talk


# Integrating 3D content
quicklook support.

AR quicklook was originally introduced in iOS 12.  All you need is a link to a usdz file.  With this attribute, and the image tag to preview.

This exact setup will work on xrOS.  Providing your users with an easy wway to put 3D objects in their space.

[[Explore the USD ecosystem]]

HTML model element.  Source, and interactive, etc.

JS api is available as well.  If you want the latest safari, you can enable this on any platform.

WebXR developer preview.  Fully immersive scenes on the web.  Based on WebGL.  various built-in support.  make it fully immersive by requesting a webXR session.  See feature flag in advanced settings.

# Wrap up
* start experiimenting
* review interactive regions
* send feedback
[[What's new in CSS]]

# References
* https://developer.apple.com/augmented-reality/tools/
* https://developer.apple.com/documentation/arkit/arkit_in_ios/usdz_schemas_for_ar
* https://developer.apple.com/documentation/safari-release-notes
* http://feedbackassistant.apple.com

