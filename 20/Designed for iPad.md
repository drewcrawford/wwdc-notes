#ipados

# Mac
Desktop and portable
keyboard and mouse input
includes advanced features
# iphone
quick, on the go
multi-touch
only the most important features

What about iPad?

# Layout
## Flatten your navigation
Avoid fullscreen transitions in favor of updating part of the screen.  e.g., photos is hierarchical

## Show more content
Small tweaks to density can make your app seem more efficient / powerful.
300% more files per screen!

## Stay in context
Show more context rather than use modaly transitions.
Would it work better if your controls were in the same level rather than in overlays?

## Create immersive experiences
# Inputs
## Start with touch
Your iPad apps should always start with touch interactions, even if you support extra inputs.

## Adopt system features
add keyboard shortcuts for all the common actions in your app.

[[designing for the ipados pointer]]

scribble

## Combine multiple inputs
Command-tap, option-tap, etc.

## Always stay responsive
E.g. you can tap->scrub in a menu.  Rather than waiting for the animation to appear.

Or you can scroll outside a menu to instantly dimiss it, in addition to cancelling the popover/menu.

# Sidebars
* app layouts optimized for iPad
* Modal and non-modal editing
* dragndrop
* collapsable sections
* overlay presentation
* fluid swipe gestures
* 3-column layouts

## Add a sidebar to your app

Flat navigation - similarly-weighted toplevel content options.  Tab bar.
Hierarchical navigation - organizing deep levels of content

"Unless your app is something immersive like a game", it usually follows into one of these two categories.

### flat

Tabs are good for flat navigation.  We suggest taking existing tabs, and putting them at the top.  Note that you should continue to use tab bar in compact layouts.

Generally, put user content below navigation.  Possibly nested under a collapsible header.  Resist the urge to put in your entire app.

Good idea to include "add" buttons at the bottom of each section.

### hierarchical
Surface the top items at the top.

Below that, add shortcuts to the most important places in your app.

Keep in mind, the sidebar is not good at browsing deep levels of hierarchy.  Instead, use the content area for navigating hierearchy.

### tips

* Don't use sidebars in compact width size classes, including iPhone.
* Convert to tab bars or table rows
* Don't mix sidebars and tab bars in the same view.
* Don't surface your entire app in the sidebar.

* Keep primary navigation at the top
* convert sidebrs to tab bars in compact width layouts
* use outlined glyphs
* Include user's most important content
* support edit mode
* support drag-and-drop

# Toolbars

## Tips
Place toolbar buttons at the top in iPadOS.  
In the compact-width, move toolbar items to the bottom

 # wrap-up
 * Design specifically for the iPad
 * Flatten navigation to fill the display
 * support touch, pencil, keyboard, trackpad
 * use sidebars for fast navigation
 * Place actions in the navigation bar

[[Design with iOS pickers, menus, and actions]]
[[Build for iPad]]

 
