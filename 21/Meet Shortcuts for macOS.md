# What's new
All your shortcuts synced from iPhone.  Gallery, etc.

Create your own shortcuts: use shortcuts editor.

Entire mac app in swiftUI to allow unified codebase.

Any app on the mac can provide access to shortcuts.

AppleScript, automator.

Two new automation types: focus and sound recognition, along with new actions.

File provider support => various file actions automatically work.
SiriKit intents => automatically expose those capabilities to shortcuts

Sharing: Easier to distribute.  New file format.  Private sharing.

## Distribution
Shortcuts are now easy to download.  Can be distributed on your website or in your app.  Notarized.

## Files
Useful if you need to distribute shortcuts outside of iCloud.  Notarized.

## Private sharing
New mode for sharing with contacts, or saving personal backups.  Shortcut files are signed (with the identity of whoever sent it)

# Mac autmation technologies
Long history of automation tech.  AppleScript, shell script, etc.

Shortcuts have full support for applescript and shell script.  

**Shortcuts is the future of mac automation**.  To make the transition, we built a migration tool that can convert most automator workflows.

Open workflow file in shortcuts app, that's it.  Your workflow is turned into a shortcut.


# How your app fits in
Expose capabilities with actions.  By exposing actions, people can use functionality faster and create workflows.  

Opening people up to use your features faster and in more places than ever before.  Actions can be used in multi-step shortcuts.  Enable people to use apps together.

[[Design great actions for Shortcuts, Siri, and Suggestions]]

# Building actions
Just like on iOS, use the Intents framework (SiriKit) #sirikit 

Types (nouns) + actions (verbs) = ? (intents)

## Building an intent
1.  Create intent definition file, add to app target
2.  Define each type
3.  Define each property
4.  Define each intent.

Put required parameters in the summary.  If you have more parameters, you can leave them out.  User can expand action and edit those.

5.  Xcode generates intent classes
6.  I can launch shortcuts app to see the new action.  It won't work yet, but it's there.
7.  App dispatch intents to handlers.  Apps need to decide which process to handle the intent.  In-app?  Or in extension?

### In-app
* Delivered to your app delegate
* Manipulate app state
* Higher resource limits

*Most apps should start with this*

In macOS monterey, you can use NSApplicationDelegate.

### Intents extension
* separate process
* lightweight and fast
* App itself won't have to launch

Override `handler(forIntent:)`

[[Empower your Intents]]

### Demo

```swift
import SwiftUI
import Intents

@main
struct SouperTaskApp: App {
    @NSApplicationDelegateAdaptor(AppDelegate.self) var appDelegate
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

class AppDelegate: NSObject, NSApplicationDelegate {
    func application(_ application: NSApplication, handlerFor intent: INIntent) -> Any? {

    }
}
```

4 different types of method
* resolve => each parameter.  Check if the parameter is valid
* provide options => Any parameter with 'dynamic options' setting enabled.  
* confirm => ensure everything looks good.  ex, check network connection is reachable
* handle => do what the intent is telling you to do

#### resolve

```swift
class IntentHandler: NSObject, CreateTaskIntentHandling {
    func resolveTitle(for intent: CreateTaskIntent, with completion: @escaping (INStringResolutionResult) -> Void) {
        guard let title = intent.title, !title.isEmpty else {
            return completion(.needsValue())
        }
        return completion(.success(with: title))
    }
    
    func resolveDueDate(for intent: CreateTaskIntent, with completion: @escaping (CreateTaskDueDateResolutionResult) -> Void) {
        guard let dateComponents = intent.dueDate else {
            return completion(.needsValue())
        }
        return completion(.success(with: dateComponents))
    }

    ...
}
```


Add a custom validation error.  Code `invalidDate`, template (error message).

App can return unsupported result with a custom validation error, which will display a message.

#### provide options
#### confirm

#### handle
```swift
class IntentHandler: NSObject, CreateTaskIntentHandling {
    func handle(intent: CreateTaskIntent, completion: @escaping (CreateTaskIntentResponse) -> Void) {
        let title = intent.title!
        let dueDate = intent.dueDate!
        
        let task = createTask(name: title, due: dueDate)
        
        let response = CreateTaskIntentResponse(code: .success, userActivity: nil)
        response.task = task
        completion(response)
    }
}
```


# Platform considerations
* Apps built with mac catalyst
* Document-based apps
* Cross-platform apps
* Running a shortcut from inside your app

## Apps built with mac catalyst
* All shortcuts APIs from iOS are now available
* If your app is already available on the mac, make sure to compile intents code from iOS.  

## Document-based app
* Improved file actions and file parameters
* Consider creating file-oriented actions for document-based apps

[[Discover built-in sound classification in SoundAnalysis]]

## Cross-platform apps
Deploy the same intents in both copies of your app.  Allows people to build shortcut on one platform and work the same way on another.

Compile the same exact intent definition for both apps, and make sure intents share the same name and parameters.

Intents with the same name will only be shared for same developer team.

On iOS 15, as long as two apps are from the same developer and use the same intent name, shortcuts will transfer from one to the other.

## Running inside your app

* Shortcuts scripting interface
* Command line

Communicate via scripting with "Shortcuts Events"

```applescript
tell application "Shortcuts Events"
	run the shortcut whose name is "Make GIF"
end tell
```

Using ScriptingBridge,
```swift
import ScriptingBridge

@objc protocol ShortcutsEvents {
    @objc optional var shortcuts: SBElementArray { get }
}
@objc protocol Shortcut {
    @objc optional var name: String { get }
    @objc optional func run(withInput: Any?) -> Any?
}

extension SBApplication: ShortcutsEvents {}
extension SBObject: Shortcut {}

guard 
    let app: ShortcutsEvents = SBApplication(bundleIdentifier: "com.apple.shortcuts.events"),
    let shortcuts = app.shortcuts else {
    print("Couldn't access shortcuts")
    return
}

guard let shortcut = shortcuts.object(withName: "Make GIF") as? Shortcut else {
    print("Shortcut doesn't exist")
    return
}

_ = shortcut.run?(withInput: nil)
```

Sandbox apps need to add an entitlement `com.apple.security.scripting-targets`

In order to list existing shortcuts need `run-targets`

CLI tool: `shortcuts`.  

# Wrap up
* Shortcuts is now on Mac.
* Contribute to the ecosystem by adding actions
* Allow peopel to be creative with your apps

* https://developer.apple.com/documentation/sirikit/dispatching_intents_to_handlers
* https://developer.apple.com/documentation/sirikit/offering_actions_in_the_shortcuts_app


