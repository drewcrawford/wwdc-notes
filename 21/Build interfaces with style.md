# Canvas
Redesigned canvas bottom bar
Clicking a different group expands it and collapses the prior group.

Drag/drop scenes.
Constraints are selectable (all constraints in group).  Modify constants for all constraints.

# Button styles
IB supports authoring buttons with new preset styles, including
* Plain
* gray
* tinted
* filled

Button styles support further customization, etc.

[[Meet the UIKit button system]]

## Demo

When a non-default style is selected, a button will be opted into the new button system.  dynamic type, etc.

New properties appear enabling customization.
* Subtitle
* alignment
* fg/bg color
* positioning
* background configuration
* corner style

## Filled style
Uses app tint to specify color

## styles
corner radius.  'dynamic' which provides a great-looking corner radius that scales with dynamic type.

Fixed remains the same as sizes change.

Both styles have customizable corner radiuses

small, medium, large, and capsule.

Button size: 
* small
* medium
* large

Button's image to `?` symbol
Set image padding.

Advantage of using IB is trying out changes quickly.

If more customization is desired, bg configuration.

Pop-up button.  Clicked to display menu with a list of items.  Single items can be selected from that menu.  Evidently this is a control in Library.

Pop-up buttons support styles.  Menus nested underneath them in outline view.

Note that on catalyst, styles map.

Menu items must be connected to actions for display at runtime.

# Table content styles

Two-finger double-tap to focus in.

Added another TV row.  Title labels should stand out.  New TV content configurations.

* Grouped header
* Value cell
* Grouped footer

DT for free.

Using one of the new styles, I can change image padding from the inspector.

Cells have larger, clear title labels.  Cells could do with some more color.




# Hierarchical symbols
Hierarchical symbols.  Symbols can have multiple layers.  Specify on a layer-by-layer basis.

* Hierarchical => specify primary layer.  Other layers are reduced alpha versions
* Palette => independently set colors

[[What's new in SF symbols]]

Varied opacities give it the emphasis that I want.  

> A fresh new look


# Accessibility
Description label uses subhead text style.  Supports dynamic type.

New in IB, you can preview how your iOS scenes react to accessiblity settings.

Canvas has new accessibility options.  Text sizes, frame values, etc.  Can preview in canvas.

Makes the label wraparound to as many lines as needed.  Label no longer clipped.

DT fits great across type sizes.  Iterate faster and stay within IB.

Use hierarchical rendering mode.  Clicking through to detail view, description fits in great.  

# Wrap up
* Canvas
* button styles
* Table content styles
* Hierarchical symbols
* Accessibility

New layout with content configuration.

REfresh, stylize, modernize your apps in IB

[[Meet the UIKit button system]]
[[What's new in SF symbols]]

