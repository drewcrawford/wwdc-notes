#swiftui  #appkit 

In macOS monterey, shortcuts came to macOS.  Uses swiftui on the mac.  Helps customize the experience for the platform while sharing common views with apps on iOS and watchOS.

How you can start adopting swiftUI in your mac app.

# Host SwiftUI in AppKit
In shortcut, main window contains appkit split VC.  


```swift
struct SidebarView: View {
    @State private var selectedItem: SidebarItem
    
    var body: some View {
        List(selection: $selectedItem) {
            ...
            Section("Shortcuts") { ... }
            Section("Folders") { ... }
        }
    }
}

enum SidebarItem: Hashable {
    case gallery
    case allShortcuts
    ...
    case folder(Folder)
}
```

```swift
let splitViewController = NSSplitViewController()

let sidebar = NSHostingController(rootView: SidebarView(...))
let splitViewItem = NSSplitViewItem(viewController: sidebar)
splitViewController.addSplitViewItem(splitViewItem)
```

NSHostingController.  Passed in as the root view of the hosting controller.  Sicne a hosting controlelr can be used as any other VC, we configure it as a split view item.  Then add that to the split view controller.

selection model.

```swift
class SelectionModel: ObservableObject {

    @Published var selectedItem: SidebarItem = .allShortcuts

}

// AppKit Window Controller
cancellable = selectionModel.$selectedItem.sink { newItem in
    // update the NSSplitViewController detail
}
```

swiftUI can reload the view when the state stored in the model changes.  Stores which sidebar item is currently selected.

When someone changes sidebar, model shows a new page.

# Collection and table cells
An iconic swiftui view built to implement a shortcut.

NSCollectionView.  Each cell view is recycled as you scroll, showing different content over time.

* ADding/removing subviews
* Reuse hosting views

In a collection/tableview with lots of tiems, each cell view is recycled.  To make srue cell reuse is performant, avoid adding/removing subviews from cells as the user scrolls.  Use a single hosting view and update it with a different root view when cell's content changes.

```swift
class ShortcutItemView: NSCollectionViewItem {
    private var hostingView: NSHostingView<ShortcutView>?

    func displayShortcut(_ shortcut: Shortcut) {
        let shortcutView = ShortcutView(shortcut: shortcut)

        if let hostingView = hostingView {
            hostingView.rootView = shortcutView
        } else {
            let newHostingView = NSHostingView(rootView: shortcutView)
            view.addSubview(newHostingView)
            setupConstraints(for: newHostingView)
            self.hostingView = newHostingView
        }
    }
}
```

Since cells are created before being configured with content, the hostingView will start as nil.  `displayShortcut` called by datasource.
Lifecycle
1.  Cell initialized with no subviews.  
2. First time `displayShortcut` is called, hosting view is created with shortcut view to display
3. Creates VStack, image, spacer, etc.
4. If the cell is scrolled offscreen, it might be dequeued
5. A new shortcut view is created and given to the hosting view

# Layout and sizing
* intrinsic size bzased on swiftUI layout
* views have min/max size
* Add constraints to surrounding views
* Use frame modifier to override

Window sizing
* windows have min/max sizes
* SwiftUI updates these for you when topmost view
```swift
viewController.present(NSHostingController(rootView: ...), 
    asPopoverRelativeTo: rect, of: view, 
    preferredEdge: .maxY, behavior: .transient)
```

popovers sized based on content when presented modally.  ex, place into an appkit popover.

or present as a sheet
```swift
viewController.presentAsSheet(NSHostingController(rootView: ...))
```

modal window
```swift
let hostingController = NSHostingController(rootView: ModalView())
hostingController.title = "Window Title"
viewController.presentAsModalWindow(hostingController)
```

window is sized to fit the content.

* New NSHostingSizingOptions API
* sizingOptions property on NSHOstingView and NSHOstingController
* Customize automatic constraints
by default, we create constraints for
```swift
hostingController.sizingOptions = [.minSize, .intrinsicContentSize, .maxSize]
```

you can modify these by changing the property.

Can enable `.preferredContentSize` option.

# Responder chain and focus
These commands include cut, copy, paste, and others.  We implemented a few of our own custom menu items as well for moving actions up and down.

Focused responder = first responder.
sleector is sent to first responder.  Then set to each next responder until something handles the selector or it reaches the app.

Equivalent to first responder in swiftUI is *focused view*.
`.focusable()` allows focus on view
some views already focusable

```swift
Image(...)
    .focusable()
    .copyable { ... }
    .cuttable { ... }
    .pasteDestination(payloadType: Image.self) { ... }
```

Easy way to let people transfer data in and out of the app.
`.onMove` and `.onExit` to handle arrow and escape keys.

Handle any of the common selectors from appkit, or your own custom selectors defined in your app.

```swift
struct ShortcutsEditorView: View {
    var body: some View {
        ScrollView { ... }
            .onMoveCommand { moveSelection(direction: $0) }
            .onExitCommand { cancelOperations() }
            .onCommand(#selector(NSResponder.selectAll(_:)) { selectAllActions() }
            .onCommand(#selector(moveActionUp(_:)) { moveSelectedAction(.up) }
            .onCommand(#selector(moveActionDown(_:)) { moveSelectedAction(.down) }
    }
}
```

When testing focus/keyboard navigability, open keyboard system settings and test with "full keyboard navigation" both on and off. M any controls only focusable with that enabled.

* more APIs in swiftUI for focus
* Use FocusState and `.focused(_:equals:)` to control flocus
[[Direct and reflect focus in SwiftUI]]


# Host AppKit in SwiftUI

You may need to host appkit views too.  One example, inside SwiftUI shortcuts editor, where there's an embedded applescript editor.

SwiftUI provides two representable protocols
* NSViewControllerRpresentable
* NSViewRepresentable

* coordinator for delegation (implementing appkit delegates?)

1.  Hosted view is initialized.
2. Maek coordinator
	3. Optional, but define your own type and return it if you need it for delegation or state management
4. 1 instance of the coordinator will stay around for lifetime fo the view
5. either makeNSView or makeNSViewController is called
	6. Context contains coordinator if any.  Maybe assign the coordinator as view's delegate?
7. Once the view has been created, the `updateView` method will be called whenver SwiftUI state or environment changes.
8. Update any properties or state stored in the appkit view to keep it in sync with the surrounding swiftui state/environment
	9. May be called often, make minimal changes.  Only reload affected part of the view
10. When swiftUI is done, it will be dismantled.  Hosted view and coordinator will both be deallocated.  Representable protocols give you an optional method to implement to clean up state.

```swift
class ScriptEditorView: NSView {
    var sourceCode: String
    var isEditable: Bool
    weak var delegate: ScriptEditorViewDelegate?
}

protocol ScriptEditorViewDelegate: AnyObject {
    func sourceCodeDidChange(in view: ScriptEditorView) -> Void
}
```

View can be disabled to prevent changes from being made.  Script editor also has a delgate, notified any time somebody modifies the sourcecode.

When hosting an appkit view, consider where the view will be placed in swiftui and what data needs to be pased in/out

In shortcuts, this view is placed into a container view next to the compile button.  Handler needs to access sourcecode entered into the view.  sorucecode is stored in swiftUI using the state property wrapper.  Representable reads/writes to the state.

Create a type conforming to `NSViewRepresentable`.

```swift
struct ScriptEditorContainerView: View {
    @State var sourceCode: String = ""

    var body: some View {
        VStack {
            CompileButton { compile(code: sourceCode) }
            Divider()
            ScriptEditorRepresentable(sourceCode: $sourceCode)
        }
    }
}
```

Now implement `makeNSView`.  How to create a new instance of the view, one time setup, etc.

Coordinators have same lifetime as the hosted view.  Properties you add to the coordinator need to remain up to date as representable changes.  

```swift
struct ScriptEditorRepresentable: NSViewRepresentable {
    @Binding var sourceCode: String

    func makeNSView(context: Context) -> ScriptEditorView {
        let scriptEditor = ScriptEditorView(frame: .zero)
        scriptEditor.delegate = context.coordinator
        return scriptEditor
    }

    func updateNSView(_ nsView: ScriptEditorView, context: Context) {
        if sourceCode != scriptEditor.sourceCode {
            scriptEditor.sourceCode = sourceCode
        }
        scriptEditor.isEditable = context.environment.isEnabled
        context.coordinator.representable = self
    }

    func makeCoordinator() -> Coordinator {
        Coordinator(representable: self)
    }
}

class Coordinator: NSObject, ScriptEditorViewDelegate {
    var representable: ScriptEditorRepresentable

    init(representable: ScriptEditorRepresentable) { ... }

    func sourceCodeDidChange(in view: ScriptEditorView) {
        representable.sourceCode = view.sourceCode
    }
}
```

# Next steps
* start adding SwiftUI to your app
* Consider sidebar, collection viwe cells
* Handle sizing, commands, focus


* https://developer.apple.com/forums/tags/wwdc2022-10075
* https://developer.apple.com/forums/create/question?&tag1=22&tag2=239&tag3=413030
* https://developer.apple.com/documentation/SwiftUI/Other-UI-Framework-Views-Displayed-by-SwiftUI
* https://developer.apple.com/documentation/SwiftUI/SwiftUI-Views-Displayed-by-Other-UI-Frameworks
