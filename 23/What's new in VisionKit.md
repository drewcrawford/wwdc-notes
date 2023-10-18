Discover how VisionKit can help people quickly lift subjects from images in your app and learn more about the content of an image with Visual Look Up. We'll also take a tour of the latest updates to VisionKit for Live Text interaction, data scanning, and expanded support for macOS apps. For more on VisionKit, check out “Lift subjects from images in your app" from WWDC23.

Last year, we added live text support.
* live text
* translation
* QR
* etc
* camera
* text types
* QR/barcodes

[[Add Live Text interaction to your app]]
[[Capture machine-readable codes and text with VisionKit]]

# Subject lifting
simple to integrate.  In fact, there's a good chance you're already finished.

ImageAnalyzer.Configuration - nothing special
subject lifting analysis is handled sepasrately after interaction is complete.  For iOS, after onscreen for a few seconds.  macos - first time the menu appears.  Don't need to handle the case of the user swiping through many photos.  Just ensure you have an appropriate interaction type set ex automatic.

| foo               | subject lifting | text | data detectors |
| ----------------- | --------------- | ---- | -------------- |
| .automatic        | yes             | yes  | yes            |
| imageSegmentation | yes             | no   | no             |
| automaticTextOnly | no              | yes  | yes               |
[[Lift subjects from images in your app]]

# Visual lookup
pets, nature, landmark, art, media
new: food, products, signs&symbols.

select languages only.

how does it work?  Two-part process.  Initial processing on device.  If the visual lookup type is present in analyzer ocnfiguration, we locate teh bounding box of results and their domain.

what actions you need to take to add to your app.  Two different ways.
* subject lifting - if the lifted subject contains 1 and only 1 visual lookup result, look up option is offered.
modal interaction -badges are placed over search reuslts.  Move to corner if they leave the viewport.  

note that you can't select text or datadetectors.  This might be used with a button to get in/out of the mode.  ex, quicklook uses the info button.
# Data scanner and live text
Optical flow
currency support

optical flow - enhance text tracking for live text experiences.  

Optical flow comes for free wwith DAtaScannerViewController but only text, not MR codes.

scan for text - specific text content type set.

ensure high framerate tracking is enabled - the default.

if your usecase allows for this configuration, optical flow tracking enhances even further. 

Data scanner has a new option, allowing users to find/interact with mentary values.  Set the text content type to `.currency`.

Get currency symbol in current locale.  Loop through each of the items and grab the transcript.  If the transcript contains the currency symbol I'm interested in, I acn update the  total value.  So in this way we can sum all values in a list.

## live text
more regions.  Expanding supported languages.

in iOS 16 we supported lists.  Easily copy/paste lists into an app that understands lists like notes.  Numbers, bullets, etc.

Now we offer the same support for tables.  

context-aware data detectors.  Data detectors and their visual relationships are used when adding contacts.Now can add a contact from a business card or flyer.

VisionKit has new text APIs.

Last year, you could get entire text content using `.transcript`.  Based on your feedback, you now have plain/attributed text, selected ranges, etc.

`textSelectionDidChange`.  Update your UI as appropriate.

Now easy to add features that rely on what the user has selected.


# expanded platform support
rolling out catalyst support from live text.  

catalyst adoption is simple.
* live text
* subject lifting
* visual lookup
* QR codes are unavailable in catalyst or native.
* but leaving `.machienReadableCodes` is fine, and is a no-op.
[[Extract document data using Vision]] if you need this functionality on the mac.

native API.
ImageAnalyzer and ImageAnalysisOverlayView

ImageAnalyzer - identical to iOS.  Exception of machineReadableCodes is a no-op.

on iOS we use ImageAnalysisInteraction, a UIInteraction added to a pre-existing view.  Does not exist on mac.

In this case, we use the ImageAnalysisOverlayView - subclass of NSView.  Need to add this in the view hierarchy above the image content.  Consider adding a subview of the NSImageView.  Generally simpler and easier to manage.

Contents rect
doesn't host or render your content, needs to know where the content exist.

Unit coordinate space - top-left origin.
top-left 0,0.  Bottom-right 1,1.

Using NSImageView, set tracking image property on the overlay view and we calculate all this automatically.

Contents rect can be provided via delegate method, also `setcontentsRectNeedsUpdate`

## contextual menus
huge part of the mac experience.  Easy to add vision-kit provided functionality to your menus.

overlayView:updatedMenu:for:at:.  Just return the menu you want.

The default implementation returns the visionkit menu.  Consider adding your own menu, or take items from this menu and add them to yours.

struct available that contains all tags, ImageAnalysisOverlayView.MenuTag.

Remember, items will only be available if they are actually valid.  So the copy item might not be in there, etc.

Customize these items however you wish, ex change the title.  These are recreated each time, don't worry about it.

you can insert your items into `.recommendedAppItems`.  Using this index is optional and not required.

note we use the rightclick as a trigger to handle subject analysis.

visonkit sets itself as a delegate for any menu you use.

if you were previously using delegate callbacks, visionkit provides its own callbacks.

quick tip: if you're in a situation... may not come from visionkit.  Keep your existing implementation around!  

consider calling your overlayView delegate implementation call NSMenuDelegate implementation.  But ensure this makes sense for your app.

# Wrap up
* subject lifting
* visual lookup
* expanded platform support


# Resources
* https://developer.apple.com/documentation/visionkit
* 