# Menu
Now show a menu from any button.  Might recognize from context menus in iOS 13.
In iOS 13, menus were represented with an actionsheet / popover.

But upon appearing, they dim the background.  Larger screens like iPad are a problem.
Actions are quite limited, e.g. can't select.  And on iPhone, have to move finger a long way.

New menus appear where you tap, so less movement.

Each action has a label on the left and an icon on the right, SMSymbol or custom image.
Title
Separator

Don't need to cancel.  Tapping outside the menu has that effect.

Menu automatically responds to DT, voiceover, etc.

## Disambiguation
Start from a clear action and once you select this action, a menu is shown to ask a more specific question.
* e.g., add button.  Add what?
* Notes, add image.  Document, take photo etc
* Done button to save.  Save video / save as new clip
## Navigation
* Tap and hold back button to show a list of things visisted
## Selection
* Can have a checkmark next to the button, like "sort options".  
## secondary options
* More button in iCloud Drive 

## Problems
Hiding all actions in a menu is not something we encourage.  Finding the right balance between primary/secondary options can help you decide which ones go in a menu vs which ones stay prominent.
In Messages, we kept "compose" because it's an important action.
Tap vs tap-and-hold.
Destructive actions -> We want to make sure there's enough friction.  Note that confirmation needs to be in a different place than the initial action.  I will have to move my finger down to the action sheet.
For destructive actions outside a menu, we recommend stickign with action sheets.

## summary
* replace action sheets and popovers
* use for
	* disambiguation
	* navigation
	* selection
	* showing secondary options
* confirm destructive actions with action sheets or popovers
* easier parity between iPad and Mac
# Date/time picker
In iOS 14, we replaced the picker wheel.  Makes it easier to select date/time, with any kind of input.

Work well if you can show inline with your view.  If it's difficult, can show "compact".

When you ask UIKit for a datepicker in compact mode, we give you a button.  

Improved way to pick date/time
Show inline
use compact when inline isn't possible
better parity between iPad/Mac

# Color picker

* grid
* spectrum
* RGB value
* dropper.  Tap the pippette in the top-left corner.

Color you select will appear in the bottom - left.  

* New component in iOS 14
* 4 ways to select colors
* Palette across apps
* Better parity between iPad and Mac

