#uikit 

Last year, we did pull-to-dismiss for sheets (non-full-screen)

[[Modernizing your UI for iOS 13 - 19]], specifically 9:45.

Use sheets in new way

* half-screen?
* dimming view
* Non-fullscreen color picker
* Popover in regular and sheet in compact

Sample app.

https://developer.apple.com/documentation/uikit/uiviewcontroller/customize_and_resize_sheets_in_uikit

# Getting a sheet
`UISheetPresentationController`.

GEt one off a VC `vc.sheetPresentationController`
.

* non-nil instance as long as the presentation style is `formSheet` or `pageSheet` (which it is by default)


# Detents

## What are detents?
Fully expanded frame.  We've exposed 2 system defined detents.

* medium -> about half height
* large -> Fully expanded sheet

`sheet.detents = [.large()]`

If you set this to `.medium(), .large()`, you get a sheet that is resizable between these two.

Can set to just `.medium()` which presents medium height alone.

It would be nice if I could show my library of photos and my postcard at the same time.

1.  `if let sheet = pickersheetPresentationController { detents = ...}`
2.  Don't dismiss

Because my detents array includes large intents, can drag bar to resize to full-height.

Scrolling scrollview will also expand the sheet.  Progressively disclosre advanced actions.

How do disable expand on scroll?

`sheet.prefersScrollingExpandsWhenScrolledToEdge`.

But in large mode, we dont' dismiss so it's not obvious anything happens.  So I want to resize to medium on tapping a photo.  Achieve this by programmatically changing detent.

`sheet.animateChanges` block.  This will animate the sheet down with standard animation curve.  And other sheets as well.

Remove dimming view.  `smallestUndimmedDetentIdentifier`.  By default this is `nil`, but you can remove by setting to smallest identifier where you don't want dimming, e.g. `medium`.   Dimming still fades in for large.

Advanced non-modal experiences.  Can interact with both sheet and also content outside the sheet.  Sort of a SXS multitasking for iPhone.

Sheet grows automatically to account for the keyboard.
# Other options
In iOS 13, we made all sheets fullscreen in landscape.  Nwo they're available in alternate appearance.  Only attached on bottom edge.
`preferEdgeAttachedInCompactHeight=true`.

If you'd like to do the narrow email compose look, use `widthFollowsPreferredContentSizeWhenEdgeAttached = true`

`prefersGrabberVisible`.  Often isn't necessary but if it's not obvious that it's resizable, grabber can be indicator of resizability.

Customize corner radius.  

# Adaptation from popover

Often a popover is wanted on iPad.

1.  `modalPresentationStyle = .popover`
2.  Instead of grabbing `sheetPresentationController` (it will be nil in this style), grab `popoverPresentationController`
3.  `barButtonItem` source
4.  `adaptiveSheetPresentationController`.  Configure just like before.

Now when I tap a button, picker appears in popover.  If I resize to compact, it adapts to a medium-height sheet.

To get the adaptive sheet, I need to read the `picker.popoverPresentationController.adaptiveSheetPresentationController` to resize back.

# Wrap up
* Adopt medium sheets
* Replace custom cards

