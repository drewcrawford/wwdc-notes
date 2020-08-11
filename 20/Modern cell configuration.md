[[advances in collectionview]]

#uikit

# View configuration
`.defaultContentConfiguration()` -> returns a fresh configuration

Basically, this seems to be a world where apple invented their own `State` struct (is it a struct) and has discovered this pattern for the first time.

Also works for tableviewheaders/footers?

Standard cell layouts and appearances are available as independent pieces that plug into any cell/view that supports them.

This also seems to separate "delegate" things (like selection) and "content" things (like title/image etc.)

Configurations are composeable – they can be used with any type of cell or view that supports them.

Background configurations vs content configurations

## Background configurations
color, effect, stroke, insets, etc.

## List content configuration
image, text, secondary text, layout metrics, etc.

## Configurations are lightweight
value types.

Always start with a fresh configuration.
Dont' worry about the old state.  UIKit efficiently updates views as needed.

Automatic animations and transitions
Single source of truth.
Maximum performance - we're able to implement many internal performance optimizations under the hood.

# Configuration state

REpresents the various inputs that you use to configure your cells and views.

Traits, states, custom states.

If you use a viewmodel to populate your cells with content, you can think of the viewmodel as a custom state as well.

All this comes together in a "configuration state".

## view configuration state
* trait collection
* highlighted, selected, disabled, focused
* custom states

## Cell configuration state

* trait collection
* highlighted, selected, disabled, focused
* editing,swiped,expanded,drag/drop
* custom states

## updates

`.automaticallyUpdatesContentConfiguration` (also `backgroundConfiguration`) - this turns on asking the configuration to update itself for a new state.  e.g., the cell is selected, what do we do? 

Very different than targetaction.

If you want to customzie the appearance for states, you can disable automatically updating configurations.

Override point to configure cell for new state - on the cell.  Override in the cell subclass.

## Color transformer

Takes in one color and returns a different color.

# Background and content configurations.

Cells apply a default *background* configuration automatically.
Get a default *content* configuration using `defaultContentConfiguration()`

Note that you can request the default configuration irregardless on 
`UIBackgroundConfiguration` or `UIListContentConfiguration` etc.
# Layouts and self-sizing

Content configurations are designed to be used with self-sizing cells.

Give cyou control over the layout margins and various padding properties.  Don't enforce a fixed height.

# Image reserved layout size
This is the size we use for the image, as a layout size (not the content size).
Text will be positioned relative to the layout size, image can extend into the content size.

# Adopting configurations

Mutually exclusive with `backgroundColor`, `backgroundView`.
Content configurations repalce UITableViewCell properties
* `imageView`
* `textLabel`
* `detailTextLabel`

These will be deprecated in a future release.  

# Using configurations with custom views

Create a list content view and add it alongside custom views.
Create or updaate this view using the configuration, and then add it as a subview alongside your own custom views.

Because the ListContentView is just a regular UIView, you can use it by itself anywhere, even outside a collection/tableview.

Even when you're building a compeltely custom view hierarchy, you can still use system configurations to help.  You can use them as a source of default values for fonts, margins, etc., even if you never apply the configuration itself.

Build a custom content configuration type.  And a paired content view class that renders it.  