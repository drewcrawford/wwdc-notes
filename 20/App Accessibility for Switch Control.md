We often talk about visual accessibility,b ut less about motor.
We may need to take additional steps.

# What is Switch Control
Users with motor impairments
They use switches to navigate a cursor on.
Switches may be mounted on wheelchairs and used by head taps, etc.
Device is often mounted with the screen on, since the wheelchair is powered.

Notice how he can use different mouth gestures to move an on-screen cursor, tap, etc.

Some users may use an automatic cursor that advances through items until a user interacts with the items.

## special considerations
* timing.  A mistap can result in numerous more steps to perform a simple action.
* Navigation efficiency and grouping.  Poor grouping can result in large wait times to arrive at the desired item.
* Error-tolerance.  Some users have tremors etc.  Use confirmations.
# APIs
*Almost all Switch Control support comes for free with good VoiceOver accessibility!*

By understanding specific APIs, we can build apps targeted for switch control.

Problems with sample app
* many levels with no clear grouping.
	* Ordinarily, switch control will travel top to bottom, left-to-right.  This travels in the wrong order through a path.
	* User looking for level 30 will wait a long time
* Relies on touch to see the cards
	* switch control user might need to interact multiple times.
* Use gesture to interact
	* Possible on switch control, but require navigating into a gesture submenu.

```swift
//group items in  a single section
//note that we have contianer views for every 3 levels already
containerView.accessibilityNavigationStyle = .combined

//control ordering of elements to avoid top-bottom, left-right ordering
containerView.accessibilityElements = [ levelFourView, levelFiveView, levelSixView]
```

We added a setting in our app settings to auto-flip cards when a user gets to them.

```swift
// Following Focus API 

class CardView : UIView { 
    var orientation: CardOrientation
    
    enum CardOrientation {
        case front
        case back
    }
    
    override func accessibilityElementDidBecomeFocused() {
        self.flip(to: .front)
    } 

		override func accessibilityElementDidLoseFocus() {
        self.flip(to: .back)
    }
    
// The rest of the class…
}
```

Custom actions: Quick access to behaviors in your app.
Reduce clutter.  Improve convenience and speed

```swift
// Custom Actions API (VoiceOver uses this too)

func configureActions() {

  let pinAction = UIAccessibilityCustomAction(
      name: "Pin Card") { (_) -> Bool in
          self.setPinned(true)
          return true
      }
  pinAction.image = UIImage(systemName: "pin")
       
  let addAction = UIAccessibilityCustomAction(
      name: "Add Card") { (_) -> Bool in
          self.setSelected(true)
          return true
      }
	  //new: set an image for a custom action
    addAction.image = UIImage(systemName: "add.square")

        
	self.accessibilityCustomActions = [addAction, pinAction]
}
```
This places our actions on the top-level menu.

Other api
```swift
static var isSwitchControlRunning: Bool { get } //or notification

//there are sometimes... UI that would otherwise be static (UILabel)
//that updates when the user taps.  Here we let switchcontrol know
//that it should navigate to this item even though it looks like static text.
var accessibilityRespondsToUserInteraction: Bool { get set }
```

# Best practices
* confirm destructive behaviors or high-consequence behaviors
* No timeouts.  Even for security purposes, timeouts may be frustrating for switch control users
* Group controls.
* Don't keep private user info on screen.  Device may be mounted on wheelchairs.