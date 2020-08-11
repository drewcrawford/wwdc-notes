#ipados 

# Design Principles
Why add pointing to iPadOS?

Leaving hands on same plane as the keyboard.  Editing text requires fine manipulation.

Precision.  In some cases it's helpful to have very precise control.

Most of us are experts at using mice/trackpads, but the traditional pointer requires a lot of physical dexterity to use.  

Traditional pointer moves at pixel-level precision.  You can place a pointer at any pixel, but the underlying UI only has 3 buttons, and only cares which button you tap, not the precise pixel coordinate.  Inconsistency between the precision of the pointer and the precision required by the app.

Pointer will adjust itself to match the precision of the buttons.  Pointer snaps to button, while morphing its shape to the shape of the button.  This feedback reduces the likelihood that you'll misclick.

A traditional pointer is drawn over the whole UX, obscuring the controls you interact with.  We want to highlight the control.  Pointer actually moves "behind" the button, so the icon has the brightness level that the designer intended.

Similar precision mismatch for lines of text.  This makes it impossible to give it an ambiguous position between 2 lines.

Similar issue with creating calendar events.  
# How the Pointer Works
When you move the pointer, you're moving 2 pointers.  The one you see on screen, and an invisible one that tracks the true position.  The *model* pointer.

We use the model pointer to decide which element the pointer is hovering over.

When a pointer moves over the button, the model pointer doesn't move to the center of the button, it remains on the edge.  This is indicated with a parallax effect.  Therefore the amount of the true pointer is always based on movement on the trackpad.
The visible pointer will animate to the next button once the model pointer moves to the edge of the button's hit area.  Lifting your finger will center on the control.  Pointer auto-hides after a short delay.  If the pointer remains on screen, it might cause people to stick to trackpad rather than touch/pencil/etc.  We want people to feel like they can move between inputs.

In fact, you can use touch and pointer simultaneously.  

We've carefully tuned the acceleration curve.  

Sometimes the boost you get on the acceleration curve isn't enough to move the pointer to where you want it.  But if we made the accel curve stronger, it could make it hard to control.
On iPadOS, we added intertia.  

Magnetism scans the interface to find the control you most likely want to target. 
The moment I lift my finger from the trackpad, the pointing system calculates wher eit would have landed if it continued moving with inertia.  So we start scanning around that position, in circles up to a fixed radius.  We'll find the nearest target in the direction of your swipe, and automatically move the pointer there.

Any control in your app that supports pointer snapping gets magnetism for free.

# Pointer Effects

## Highlight effect
Use for small controls that don't have a background.  Default effect for buttons and tab bars.

Becomes the control's background.  Pointer scales content (?) and shows with a parllax effect.  Highlight added on top.  On click, the layers scale down and move to the object center.

## lift effect
Pointer transforms into the element.  App icons.  When the pointer approaches the control, the... hides behind it.  Use this for controls that have a background.

1.  specular -> added on top, shows position of the pointer.  Connects your gestures with the movement on the screen.  Strength is based on the object's size as well.  Brighter for larger objects, when the model pointer is more difficult to reason about, dimmer for smaller objects when the model pointer is more obviously constrained to a smaller area
2.  element -> scaled, 
3.  radiosity -> a sort of vibrancy-like effect
4.  shadow -> gives the illusion it's floating

## hover effect
Used for larger objects which would behave poorly if pointer morphed into the shape.
Cursor retains the default shape and remains visible on top of the object.  Examples include large buttnos, notifications in notification center, etc.  Basically things that seem "wide".

How do you use?

Try the automatic effect.  System uses rules to decide the best effect.  These rules can change in the future, so using the automatic effect is forward-proof.  `UIPointerEffect.automatic` [[Build for the iPadOS pointer]]

However, this may be inconsistent with other controls.  Avoid this discrepancy by keeping a consistent effect (and also size).  We recommend using a height of 37 points.  Also try to avoid the highlight effect around rectangular objects. 

Define a hit region that feels conmfortable.  Add about 12 pts of padding around elements that include a bezel, and around 24 points around elements that do not include a bezel (e.g input button).  An extended hit region is also useful for touchscreen

Make sure hit region extends between objects, so you don't have gaps.

A common problem with the lift effect is the shadow we add.  Because other elements might clip it, etc.  Make sure the objects that adopt lift are on top of their surrounding layers to ensure the shadow is cropped correctly.

Important you provide correct corner radius.

Can highlight objects you don't want to scale.  When tinting is turned on, it will add a special material above/below the object.  Can be dark or light, depending on system settings.

Can provide a custom shape for the pointer.  The pointer will morph from the ... into the custom shape.  Can show extra information or a change in the pointer behavior.

If we visualize the position of the pointer, we can see how the ibeam tries to stay in the center of the text field.  This is called snapping.  Increases the precision of the pointer.

*Never enable snapping if you are not providing a custom pointer.*

* try to use automatic first
* be consistent with effects and shape
* extend objects hit region
* provide correct size and corner radius
* customize the hover effect

# Pointer Appearance and Shape
## Pointer material
Pointer in iPadOS has a material that constantly adapts to the background color smoothly.  Provides optimal contrast.  When the bg color changes, so will the pointer's color.  This material maintains contrast without obscuring the background.  

When the pointer changes shape, we also adjust the color to provide good legibility to the button underneath.

When people click, the circle scales down and becomes darker.  

## Transformation
* rectangle.  `UIPointerShape.roundedRect(CGRect, radius: CGFloat)`
* arbitrary shape.  Used to indicate a change of functionality or position.  e.g., cross-shape to indicat ean area of high-level of precision.  Or triangles to indicate direction of drag movement.  Or, rotation.  `UIPointerShape.path(UIBezierPath)`

See [[Build for the iPadOS pointer]]

## How to design a custom shape?
1.  Make sure shape is simple and easy to understand
2.  Use a solid shape.  Color is constantly changing, solid shape improves legibility.  Strokes will be difficult to read.  If you must, use heavy strokes.  We use 4.5pt for cross.
3.  Reference default size, a 19pt circle.  A narrow shape may want to be bigger.  e.g. cross is 24pt.  Check transformation between 2 states.
4.  Show intention directly.  e.g., rather than showing a marker icon for highlight, show the marker tip.  This more closely resembles the i-beam.
5.  Make sure that anchor points match as well.  If your custom shape is a circle, symmetric shape, etc., use the center.  If the shape is directional, we will need the anchor point set to match the intent.  But it also means the transformation will appear offcenter.  e.g., a target shape may be better than eyedropper, because it's balanced.
6.  Prefer system standard pointer behaviors.  
7.  Similar shapes for similar functions.  So that people can apply knowledge from one area of your app to the others.
8.  Avoid creating unnecessary custom pointers.  People expect the change to be useful.  Purely decorative effects can irritate.


# Pointer Interactions
## Increasing precision
People can now interact with your app with much greater precision than previously possible.  

ex, text.  We begin selecting with character-level precision immediately.
Strive to always use a pointer that matches a specific... 

Numbers.  We can tap to select and then drag to resize column.  But with pointer input, we can expose a resize directly.  Avoiding the need to first make an imprecise selection to achieve our goal.

Neither of these examples add new functionality to the pointer, but use added precision.  Avoid features that only work with the pointer.

## Improving low-precision input
Consider places in your app where adjacent controls can be cool with pointers.  e.x., reminders.

Add a hover effect to indicate interactivity in calendar events.

## Accelerating interaction
Pointer's presence indicates intent.  We can use this to speed up interaction that previously would have required a tap.  We know that hovering over a region indicates a person is likely ot interact.

eg., books controls visibility based on hover.  Removes need of a click to hide/show.

Pointer's movement can also be a hint to update UI.  e.g., playback.  

Since we use 2-finger scrolling for scrollviews, a tap-drag can do things like faster drag-and-drop, or drag to select, whereas in a touch interface we'd have to wait for scroll GR to fail.


# Gestures
## Two-finger gestures
While one-finger gestures are generally reserved for pointing and clicking
and three-finger gestures are reserved *by the system* for e.g. multitasking
your app can use 2-finger gestures on the trackpad.

2-finger scrolling for scrollviews.  When adding gestures to the trackpad, treat them as happening relative to the pointer.  Gestures should be performed on the view underneath the pointer.

How 2-finger gestures behave in maps. Zoom/rotate.  Note that view remains anchored to the pointer.  This keeps the pointer in a fixed position relative to the map below.

Think about places in your app that allow draggint today that could be adapted to 2-finger trackpad drag.

## Secondary-click
Instantaneously perform an action that would otherwise require a long press.  By default, we use this to immediately show context menus.  