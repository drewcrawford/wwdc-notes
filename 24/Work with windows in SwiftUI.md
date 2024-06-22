Learn how to create great single and multi-window apps in visionOS, macOS, and iPadOS. Discover tools that let you programmatically open and close windows, adjust position and size, and even replace one window with another. We'll also explore design principles for windows that help people use your app within their workflows.

### 2:36 - BOT-anist scenes

```swift
@main struct BOTanistApp: App {
    var body: some Scene {
        WindowGroup(id: "editor") {
            EditorContentView()
        }
        WindowGroup(id: "game") {
            GameContentView()
        }
        .windowStyle(.volumetric)
    }
}
```

### 3:09 - Creating the movie WindowGroup

```swift
@main struct BOTanistApp: App {
    var body: some Scene {
        WindowGroup(id: "editor") {
            EditorContentView()
        }
        WindowGroup(id: "game") {
            GameContentView()
        }
        .windowStyle(.volumetric)
        WindowGroup(id: "movie") {
            MovieContentView()
        }
    }
}
```

### 3:55 - Opening a movie window

```swift
struct EditorContentView: View {
    @Environment(\.openWindow) private var openWindow
    
    var body: some View {
        Button("Open Movie", systemImage: "tv") {
            openWindow(id: "movie")
        }
    }
}
```

### 4:45 - Pushing a movie window

```swift
struct EditorContentView: View {
    @Environment(\.pushWindow) private var pushWindow
    
    var body: some View {
        Button("Open Movie", systemImage: "tv") {
            pushWindow(id: "movie")
        }
    }
}
```

### 5:34 - Toolbar

```swift
CanvasView()
    .toolbar {
        ToolbarItem {
            Button(...)
        }
        ...
    }
```

### 5:40 - Title menu

```swift
CanvasView()
    .toolbar {
        ToolbarTitleMenu {
            Button(...)
        }
        ...
    }
```

### 5:48 - Hiding window controls

```swift
WindowGroup(id: "movie") {
    ...
}
.persistentSystemOverlays(.hidden)
```

### 6:28 - Creating the controller window

```swift
@main struct BOTanistApp: App {
    var body: some Scene {
        ...
        WindowGroup(id: "movie") {
            MovieContentView()
        }
        WindowGroup(id: "controller") {
            ControllerContentView()
        }
    }
}
```

### 6:34 - Opening the controller window

```swift
struct GameContentView: View {
    @Environment(\.openWindow) private var openWindow
    
    var body: some View {
        ...
        Button("Open Controller", systemImage: "gamecontroller.fill") {
            openWindow(id: "controller")
        }
    }
}
```

### 7:46 - Positioning the controller window

```swift
WindowGroup(id: "controller") {
    ControllerContentView()
}
.defaultWindowPlacement { content, context in
    #if os(visionOS)
    return WindowPlacement(.utilityPanel)
    #elseif os(macOS)
    ...
    #endif
}
```

### 8:45 - Positioning the controller window continued

```swift
WindowGroup(id: "controller") {
    ControllerContentView()
}
.defaultWindowPlacement { content, context in
    #if os(visionOS)
    return WindowPlacement(.utilityPanel)
    #elseif os(macOS)
    let displayBounds = context.defaultDisplay.visibleRect
    let size = content.sizeThatFits(.unspecified)
    let position = CGPoint(
        x: displayBounds.midX - (size.width / 2),
        y: displayBounds.maxY - size.height - 20
    )
    return WindowPlacement(position, size: size)
    #endif
}
```

### 10:12 - Default size

```swift
@main struct BOTanistApp: App {
    var body: some Scene {
        ...
        WindowGroup(id: "movie") {
            MovieContentView()
        }
        .defaultSize(width: 1166, height: 680)
    }
}
```

### 10:49 - Setting resize limits on the movie window

```swift
@main struct BOTanistApp: App {
    var body: some Scene {
        ...
        WindowGroup(id: "movie") {
            MovieContentView()
                .frame(
                    minWidth: 680,
                    maxWidth: 2720,
                    minHeight: 680,
                    maxHeight: 1020
                )
        }
        .windowResizability(.contentSize)
    }
}
```

### 11:37 - Controller window resizability

```swift
@main struct BOTanistApp: App {
    var body: some Scene {
        ...
        WindowGroup(id: "controller") {
            ControllerContentView()
        }
        .windowResizability(.contentSize)
    }
}
```
# Resources
* https://developer.apple.com/documentation/visionOS/BOT-anist
* 