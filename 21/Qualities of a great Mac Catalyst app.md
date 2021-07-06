3 considerations of making an amaizing catalyst app.

#catalyst 

# Migrating to mac catalyst
First, have a great iPad app.  
* iPad multitasking
* Menu builder
* Copy/paste
* Drag and drop
* [[Qualities of great iPad and Iphone apps on Macs with M1]]

Bridge system behaviors like c/p and drag and drop.

You gain the ability to distribute to all macs and get access to additional APIs.

Check mac option under 'deployment info'.  Choose between scaled iPad and optimized.

## Use modern APIs
* OpenGL ES => Metal
* Addressbook => Contacts
* UIWebView => WKWebView

Check third-party dependencies
* XCFrameworks => need to provide a mac binary

* Look for compiler warnings and log messages
* Use only supported API

## Lifecycle
* Use scene lifecycle events
* Background events are more rare
	* Remember, mac catalyst app will not receive sceneDidEnterBackground as often as an iPad
	* Scenes enter the background when desktop window is minimized or closed.  Consider e.g. autosave somewhere else.
* May have zero scenes
	* All windows are closed but name remains in the menu bar

# Adopt mac idiom
* Pixel-perfect text and images
* Native controls
* Metrics will change
* Custom controls

Your app runs at 100% scale.  

[[Optimize the Interface of Your Mac Catalyst App]]

Provide both 1x and 2x assets to support all monitor resolutions.

Automatically, you get the mac style of control, but can use customization APIs that aren't available on mac controls?

Setting thumb on UISlider will appear larger than expected by default.  

Keep in mind that mac users expect AppKit style controls.  Use controls sparingly.

## Mac style controls
* What makes these controls different
* How to get specific controls

[[21/What's new in Mac Catalyst]]

Understanding these controls and where they're commonly found will help make informed choices.

Button takes on expected appearance for context.  In mac idiom, this becomes a bordered push button.

PUlldown button

```swift
button.menu = UIMenu(...)
button.showsMenuAsPrimaryAction = true
```

Single arrow indicator.  PDF in print dialog.  Displays various actions.

Make sure you have assigned a UIMenu to your button via the menu property.

`showsMenuAsPrimaryAction = true`

Pop-up buttons.  Similar ot pulldown, but have a double-arrow indicator.  Where pulldown triggers an action, a popup button is used to select one of a set of mutually-exclusive options.

Title in the button updates to reflect this selection.  

Checkboxes are used to represent a non-exclusive binary toggle and are a more friendly alternative to a switch.

Make sure that a switch has a title set.  The title property is only supported in the mac idiom.  Switch has a preferred style of automatic.  Verify at runtime whether it's a switch or checkbox using the read-only style property

# Specific things you can do
Access to more screen real estate.  Windows can be resized and shown fullscreen.  Resize your app's windows and pay attention layout. 

Show more content and controls?

Put layout performance to the test.  Do the least amount of work possible to keep work responsive during resizing.

Modal presentations and popovers.  Larger display area can make interactions available.

##  Pointer input
* No trackpad
* No scroll?

Make sure capabilities are accessible using a mouse without scroll input.  Add additional buttons or controls to make sure functionality is accessible.

Detecting keyboard modifiers on tap or pan GR can provide fast access to view's functionality.  e.g. shift-pan to zoom.

Main menu - discover great actions in your app. 

## Keyboard shrtcuts and main menu

`UIResponder.keyCommands` => `UIMenuBuilder`
Add all actions

Makes them discoverable when not currently enabled.

Organize shortcuts on iPad shortcuts overlay.  Add all actions needed to interact.

Gestures should be accessible from main menu.

Adding keyboard shortcuts will provide quicker access to these actions.

## Action routing
* First responder and focus

Since mac must rely less on direct manipulation of views, but instead select items and select actions.  First responder becomes more important.

[[Focus on iPad Keyboard Navigation]]

## Do not modify the responder chain
 Don't override nextResponder
 Leaving chain unmodified ensures that mac catalyst can route to appropriate targets.
 
 Use `target(forAction:withSender:)`
 
 ```swift
 final class MyView: UIView {
    override func target(forAction action: Selector, withSender sender: Any?) -> Any? {
        if action == #selector(Model.setAsFavorite(_:)) {
            return myModel
        } else {
            return super.target(forAction: action, withSender: sender)
        }
    }
}
```
View delegates something to a model object.

## Scenes

* One `UIWindowScene` per window.

May have windows with different functions.

### Configurations
Add to your info.plist under "Application Scene Manifest"

Create one config for each scene type.  Give each config a name and choose your scene class, delegate class, and storyboard for hwn scene is created.

Create a new detail viewer scene when a view is double-clicked.  

Define new user activity type for requesting a detail viewer scene.

```swift
let viewDetailActivityType = "viewDetail"
let itemIDKey = "itemID"

final class MyView: UIView {
    @objc func viewDoubleClicked(_ sender: Any?) {
        let userActivity = NSUserActivity(activityType: viewDetailActivityType)
        userActivity.userInfo = [itemIDKey: selectedItem.itemID]
        UIApplication.shared.requestSceneSessionActivation(nil,
            userActivity: userActivity,
            options: nil,
            errorHandler: { error in //...
        })
    }
    //...
}
```

## Responding to scene request

```swift
let viewDetailActivityType = "viewDetail"

final class AppDelegate: UIApplicationDelegate {
    func application(_ application: UIApplication, 
        configurationForConnecting session: UISceneSession, 
        options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        if let activity = options.userActivities.first {
            if activity.activityType == viewDetailActivityType {
                return UISceneConfiguration(name: "DetailViewer", sessionRole:session.role)
            }
        }
        return UISceneConfiguration(name: "Default Configuration",
            sessionRole: session.role)
    }
    //...
}
```

In our implementation, we examine if the incoming request contains user activities.

Request can contain multiple activities, but we just examine the first one.

One more thing to do.  Remember that we saved the item id for the item to be shown?  Need to set that value on the new VC.

```swift
let itemIDKey = "itemID"

final class SceneDelegate: UIWindowSceneDelegate {
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
        options: UIScene.ConnectionOptions) {
        if let userActivity = connectionOptions.userActivities.first {
            if let itemId = userActivity.userInfo?[itemIDKey] as? ItemIDType {
               // Set item ID on new view controller
            }
        }
        //...
    }
    //...
```

Using NSUserActivity to configure new scenes makes it easier to support state restoration.  If your scene delegate responds to

```swift
final class SceneDelegate: UIWindowSceneDelegate {
    func stateRestorationActivity(for scene: UIScene) -> NSUserActivity? {
        //...
    }
}
```

Your activity will be saved.  

If enabled in sysprefs, system will recreate your scenes and pass the UA object to your app delegate's configuration.

By using consistent activity types, can use the same code to select an appropriate scene configuration for new windows and during state restoration.

Modify sceneWillConnect to fall back to state restoration, if the activity is nil.  Now your app is ready to handle new scene requests, and state restoration.

```swift
let itemIDKey = "itemID"

final class SceneDelegate: UIWindowSceneDelegate {
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions) {
        if let userActivity = connectionOptions.userActivities.first ??
            session.stateRestorationActivity {
            if let itemId = userActivity.userInfo?[itemIDKey] as? ItemIDType {
               // Set item ID on new view controller
            }
        }
    }
}
```

[[Introducing Multiple Windows on iPad - 19]]

## Toolbar
Uses windows toolbars for frequently-used options for quick access.

Toolbar on catalyst does not change as VCs appear and disappear in split VC or navigation controller.

Because toolbars are strongly associated with scenes, configure your toolbar in scene delegate subclass.

Sharing button.  Adding an NSSharingServicePickerToolbarItem => use standard sharing menu.

In monterey, the button can automatically use the activity item's configuration.

Same config that "share this" on siri uses from iOS.

```swift
final class RootViewController: UIViewController {
    override var activityItemsConfiguration: UIActivityItemsConfigurationReading? {
      get { UIActivityItemsConfiguration(objects: [image]) }
      //...
    }
}
```

On catalyst, we have copy/paste in a menu before the share menu.  On iOS, integrated into the share sheet.

## Continuity camera

Automatic for UITextView (rich text).  Can take a photo and add it as attachment.

Custom view:

```swift
final class MyView: UIView {
    override var pasteConfiguration: UIPasteConfiguration? {
      get { UIPasteConfiguration(forAcceptingClass: UIImage.self) }
      //...
    }

    func willMove(toWindow: UIWindow) {
       addInteraction(contextMenuInteraction)
    }

    override func paste(itemProviders: [NSItemProvider]) {
       for itemProvider in itemProviders {
            if itemProvider.canLoadObject(ofClass: UIImage.self) {
                if let image = try? await itemProvider.loadObject(ofClass:UIImage.self) {
                    insertImage(image)
                }          
                //...
```

## Paste configuration
* Continuity camera
* Paste
* Drag/drop


# Distribution

Mac catalyst apps are mac apps
* MAS
* TestFlight
* App notarization
* XCFrameworks








