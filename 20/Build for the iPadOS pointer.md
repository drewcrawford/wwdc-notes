#ipados 

In 13.4, we added general pointing device support.

What value can a pointer add, and how does it coexist with a touch-based interface?

[[designing for the ipados pointer]]

# Updating your app for pointer support

## What just works?
* Built-in pointer effects
* `UIBarButtonItem`
* `UISegmentedControl`
* `UIMenuController`
* `UIScrollView`
* pinch-to-zoom

Collection/tableview support 2-finger panning

`UIDragInteraction`.  
`UIContextMenuInteraction` lets you invoke its menu in a new compact appearance via secondary click.

## updating your app
Start with the higher-level APIs.  

* Controls: `UIButton`, `UIBarButtonItem`.  Some offer APIs that allow you to enable the effects rather than have them enabled by default
* Interaction: `UIPointerInteraction`.  You can choose from 1 of a collection of system-vended effects to apply to your views.  
* `UIHoverGestureRecognizer` -> Direct motion

[[Handle trackpad and mouse input]]

* aim for consistency with the system
* Add pointer effects where they hadd value
* start with app chrome.  Top/bottom bars

### `UIBarButtonItem`
* works automatically
* manually enable custom view items
* Button-based custom view items enabled by default.


### `UIButton`
2-stage convenience APIs

`button.isPointerInteractionEnabled = true` to enable effect
`button.pointerStyleProvider` to customize it.



`PointerStyleProvider` -> Proposed style" determined by the system
customize and return a new style

```swift
// Enable the button's built-in pointer interaction.
myButton.isPointerInteractionEnabled = true

// Customize the default interaction effect.
myButton.pointerStyleProvider = { button, proposedEffect, proposedShape -> UIPointerStyle? in
		// In this example, we'll switch to using the .lift effect by creating a new
    // UIPointerEffect with the .lift type using the proposedEffect's preview.
    return UIPointerStyle(effect: .lift(proposedEffect.preview), shape: proposedShape)
}
```
Styles fit into 2 categories
* Content effect -> morph into a view in the app.  e.g. highlight effect applied to bar buttons.
	* `UIPointerEffect` -> visual treatment
	* `UIPointerShape` -> Shape to which the pointer will change

```swift
// Create a UIPointerStyle that applies the .highlight effect. 

// Outset the view's frame so the pointer shape has some generous padding around the view's contents.
// Note that this frame must be in the provided UITargetedPreview's container's coordinate space. 
// In the majority of cases (where the preview doesn't have a custom container), this is just the view's superview.
let rect = myView.frame.insetBy(dx: -8.0, dy: -4.0)
let preview = UITargetedPreview(view: myView)

return UIPointerStyle(effect: .highlight(preview), shape: .roundedRect(rect))
```

e.g. `UIPointerStyle(effect: .highlight(...),shape:.roundedRect(...))`
* Shape customization.  The pointer morphs into the shape and is constrained along the specified axes.  e.g. pointer behavior in text views.  Vertical beam is the shape, and vertical is the constrained axis.
	* `UIPointerShape`
	* `[UIAxis]` mask.

```swift
// Create a UIPointerStyle that changes the pointer into a vertical beam. 

let beamLength = myFont.lineHeight
return UIPointerStyle(shape: .verticalBeam(length: beamLength), constrainedAxes: .vertical)
```

# Pointer related API
## demo

# Custom UI
Focus on areas where you think they'll add utility and unique value.

Since this is an entirely custom view, we'll use `UIPointerInteraction` directly.

* Attach interaction to view
* Delegate is optional

`UIPointerStyle` -> what happens to the pointer
`UIPointerRegion` -> where the style is active

Default region covers the entire view
Applies `.automatic` pointer effect by default

This is convenient for cases where we want to apply a default effect to a view, but our quilting app is more specialized.

```swift
//requests new regions as we move within the view
func pointerInteraction(_ interaction: UIPointerInteraction, 
                          willEnter region: UIPointerRegion, 
                          animator: UIPointerInteractionAnimating) {

     // Fade out separator when entering region.
     animator.addAnimations {
          self.separatorView.alpha = 0.0
     }
}
```

Also implement `styleForRegion:` to return the crosshair icon.

## demo

## polish
* Padding around views.  A pointer regions that extends the pointer style's affected area.  Note that any regions you provide, must be within the hit-testable area of the view.  If you provide a larger area, must ensure that it hit-tests to the view.
* Larger UIPointerRegions.  Scrub through reminders by creating contiguous pointer regions.  If often makes sense to take your stuff to a higher level in the view hierarchy.  In this case, the interaction is tied to the entire view.
* Coordinate with pointer animations.  E.g. hiding separators between `UISegmentedControl`:

```swift
func pointerInteraction(_ interaction: UIPointerInteraction, 
                          willEnter region: UIPointerRegion, 
                          animator: UIPointerInteractionAnimating) {

     // Fade out separator when entering region.
     animator.addAnimations {
          self.separatorView.alpha = 0.0
     }
}
```

```swift
func pointerInteraction(_ interaction: UIPointerInteraction, 
                          willExit region: UIPointerRegion, 
                          animator: UIPointerInteractionAnimating) {

     // Fade separator back in when exiting region.
     animator.addAnimations {
          self.separatorView.alpha = 1.0
     }
}
```

* Matching hit-testing.  (?)




# Best practices
