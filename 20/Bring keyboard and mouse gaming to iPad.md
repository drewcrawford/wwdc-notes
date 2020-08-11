#ipados 

Review of various control schemes.
Direct touch
on-screen buttons.
game controllers

but what about MOBA or fps games (wasd) that don't translate well to these?

# Keyboard and mouse APIs
* `GCKeyboard`
* `GCMouse`

* connect and disconnect notifications
* register change handling blocks
* polling current state (in some cases)

These conform o `GCDevice` profile.  Slides say they conform to `GCKeyboard`, but I think it's a typo.

## GCKeyboard
* elements
* buttons
* no axes or dpads

## GCMouse
* may see 0 or more axes
* 1 or more buttons

## Keyboard event change handlers
```swift
//is there a keyboard attached?
//'coalesced' - merge all keyboard input together
if let keyboard = GCKeyboardDevice.coalesced?.keyboardInput {

  // bind to any key-up/-down
  keyboard.keyChangedHandler = {
    (keyboard, key, keyCode, pressed) in
    // compare buttons to GCKeyCode
  }
  
  // bind to a specific key-up/-down
  keyboard.button(forKeyCode: .spacebar)?.valueChangedHandler = {
    (key, value, pressed) in
    // spacebar was pressed or released
  }

}
```

## poll keyboard state
```swift
func pollInput() {
  
  if let keyboard = GCKeyboardDevice.coalesced?.keyboardInput {
	  if (keyboard.button(forKeyCode: .keyW)?.isPressed ?? false) { /* move up    */ }
	  if (keyboard.button(forKeyCode: .keyA)?.isPressed ?? false) { /* move left  */ }
	  if (keyboard.button(forKeyCode: .keyS)?.isPressed ?? false) { /* move down  */ }
	  if (keyboard.button(forKeyCode: .keyD)?.isPressed ?? false) { /* move right */ }
  }
  
}
```

**Change callbacks or polling are equally valid.**
Polling is O(1), non-blocking, non-yielding, and threadsafe.


## mouse event change handlers
```swift
if let mouse = GCMouse.currentMouse {
   //this is the change in input since the last event was received
   //it is NOT absolute screen coordinates, and that's intentional
   mouse.mouseInput.mouseMovedHandler = {
     (mouse, deltaX, deltaY) in
     // use delta to calculate your game's cursor position or other motion
   }

}
```
* no polling of the pointer's current position
* Need delta events even at screen edges and corners

What about system gestures at screen edges?

## pointer locking
If you're using `GCMouse` for a full-screen game, you almost certainly want to do this.  

This hides the pointer and locks it to just your fullscreen app, so it can't trigger system gestures.

```swift
class ViewController: UIViewController {
    
    override var prefersPointerLocked: Bool {
        return true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()        
        self.setNeedsUpdateOfPrefersPointerLocked()
    }
}
```

## detecting connect and disconnect
```swift
class ViewController: UIViewController {
  var keyboard: GCKeyboard? = nil
  var mouse: GCMouse? = nil
  
  init() {
    let center = NotificationCenter.defaultCenter
    let main = OperationQueue.mainQueue
        
    center.addObserverForName(GCMouseDidConnectNotification, object: nil, queue: main) {
      (note) in
      self.mouse = note.object as? GCMouse
    }

	//can't differentiate between multiple keyboards.
	//all keyboards are coalesced.
	
    center.addObserverForName(GCKeyboardDidConnectNotification, object: nil, queue: main) {
      (note) in
      self.keyboard = note.object as? GCKeyboard // the same as GCKeyboard.coalesced
    }

  }
}
```
# Which API when?
GCKeyboard & GCMouse vs UIKit
It depends.

|                   | GCKeyboard/GCMouse            | UIKit                                 |
|-------------------|-------------------------------|---------------------------------------|
| Full screen       | Always or mostly              | Optional or multitasking-aware        |
| UI elements       | Mostly custom                 | System provided                       |
| Pointer rendering | Custom images                 | Standard shape blending               |
| System gestures   | Intrude on gameplay           | Useful and do not intrude             |
| Input gathering   | Multi-threaded polling useful | Standard UIResponder (on main thread) |

Note that UIKit now has all the up/down etc. keyboard events so you can be fine with that for non-immersive app use.

You may find yourself using both in different modes.
# Fox2 example
```swift
//pick up the mouse motion
var delta: CGPoint = CGPoint.zero

func registerMouse(_ mouseDevice: GCMouse) {
  
  if #available(iOS 14.0, OSX 10.16, *) {
    guard let mouseInput = mouseDevice.mouseInput else {
      return
    }
          
    // set up our mouse value change handlers
    
  }
}

    weak var weakController = self

    mouseInput.mouseMovedHandler = {(mouse, deltaX, deltaY) in
      guard let strongController = weakController else {
        return
      }
                                    
      strongController.delta = CGPoint(x: CGFloat(deltaX), y: CGFloat(deltaY))
                                    
    }
            
    mouseInput.leftButton.valueChangedHandler = {(button, value, pressed) in
      guard let strongController = weakController else {
        return
      }
                                                 
      strongController.controllerAttack()
                                                 
    }
            
    mouseInput.scroll.valueChangedHandler = {(cursor, x, y) in
      guard let strongController = weakController else {
        return
      }          
      guard let camera = strongController.cameraNode.camera else {
        return
      }
                                             
      camera.fieldOfView = CGFloat.maximum(CGFloat.minimum(120,
                                           camera.fieldOfView + CGFloat(y)), 30)

      }
    }

func pollInput() {
  ...
  
  // Mouse
  let mouseSpeed: CGFloat = 0.02
  self.cameraDirection += simd_make_float2(-Float(self.delta.x * mouseSpeed), 
                                            Float(self.delta.y * mouseSpeed))
  self.delta = CGPoint.zero
        
  // Keyboard
  if let keyboard = GCKeyboard.coalesced?.keyboardInput {
    
      if (keyboard.button(forKeyCode: .keyA)?.isPressed ?? false) { self.characterDirection.x = -1.0 }
      if (keyboard.button(forKeyCode: .keyD)?.isPressed ?? false) { self.characterDirection.x = 1.0 }
      if (keyboard.button(forKeyCode: .keyW)?.isPressed ?? false) { self.characterDirection.y = -1.0 }
      if (keyboard.button(forKeyCode: .keyS)?.isPressed ?? false) { self.characterDirection.y = 1.0 }
            
      self.runModifier = (keyboard.button(forKeyCode: .leftShift)?.value ?? 0.0) + 1.0
    
  }
}
```
# Demo video

# wrap up
[[Advancements in Game Controllers]]
[[Handle trackpad and mouse input]]

