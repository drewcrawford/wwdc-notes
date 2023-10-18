#xcode #previews

You want the fastest way to test your code and experience what you're making come to life.

Make the most of previews

# What are previews
snippet of code that makes and configures a view.  top level of sourcefile.

## Basic preview - 1:30
```swift
#Preview {
    MyView()
}
```

compiled into your app, alongside app code and resources.  Because previews can access these symbols and resources, they're very flexible.

also about iterating faster. When you edit swift code, xcode
1.  examines the change you made and recompiles the minimal amount of code
2. rerun your preview

* execute code in your project
* helps you develop faster
* preview all layers of your app
	* leaf views, large views.


# Writing previews
1.  Macro
2. trailing closures of content
3. optional:
	4. name
	5. configuration

what can you preview:
* views
	* swiftui
		* return any view
		* any other views that you need, such as list, etc.
		* modifiers such as environment
		* name?
		* traits, such as orientation
	* uikit
		* VC, configure, etc.
		* UIView, NSView, etc.
	* appkit

demo
canvas mode is enabled by default, maybe turn it on.
canvas is hidden unless there's a preview defined in the file.

3 different modes in the bottom left corner of the canvas.
1.  Live or interactive mode.  Interact with view in the canvas.
2. selection or static mode.  Takes a snapshot of my view, and allows me to interact with elements in the canvas.  Moves focus to source editor.
3. variants mode.  I can pick which devices i want to see, values, color schemes, dynamic type,  etc.

## Previewing a SwiftUI view in a list - 5:05
```swift
#Preview {
    List {
        CollageView(layout: .twoByTwoGrid)
    }
    .environment(CollageLayoutStore.sample)
}
```

## Previews can have a name and configuration traits - 5:37
```swift
#Preview(“2x2 Grid”, traits: .landscapeLeft) {
    List {
        CollageView(layout: .twoByTwoGrid)
    }
    .environment(CollageLayoutStore.sample)
}
```

## Previewing UIKit view controllers and views - 5:58
```swift
#Preview {
    var controller = SavedCollagesController()
    controller.dataSource = CollagesDataStore.sample
    controller.layoutMode = .grid
    return controller
}

#Preview(“Filter View”) {
    var view = CollageFilterDisplayView()
    view.filter = .bloom(amount: 15.0)
    view.imageData = …
    return view
}
```

## Xcode can help suggest a preview - 7:08
```swift
#Preview {
    FilterEditor()
}
```

## Setting a UIKit preview to start in landscape - 11:30
```swift
#Preview("All Filters", traits: .landscapeLeft) {
    let viewController = FilterRenderingViewController()
    if let image = UIImage(named: "sample-001")?.cgImage {
        viewController.imageData = image
    }
    viewController.filter = Filter(
        bloomAmount: 1.0,
        vignetteAmount: 1.0,
        saturationAmount: 0.5
    )
    return viewController
}
```

## widgets

really highlight just how fast previews can be.  2 kinds of widgets
* widgets that use a timeline provider
	* which produce individual entries
	* preview either an entire timeline provider, or create your own timeline in the preview.


## Previewing a small widget with a timeline provider - 12:20
```swift
#Preview(as: .systemSmall) {
    FrameWidget()
} timelineProvider: {
    RandomCollageProvider()
}
```

previews snapshots each timeline entry and shows it in the canvas.  Click through or use the arrow keys.  xcode communicates with my widget, and shows the animation between the entries.

this is where a timeline of specific entires comes in handy.  I can craft the exact scenario I want to iterate on.  Use `timeline:` and return the two entries that replicate the case I want to fix.

Conveniently, I can pin the preview when I navigate away.

I can loop this transition, xcode will keep playing while I fix the code.

second kind of widget: live activities.  Nearly the same, but instead of providing a timeline provider and entires, you provide live activity attribuets, and states.

API is similar, no code example.

[[Bring widgets to life]]


# Previews in your project
How to make the most of writing previews in your project.

## Previewing in Libraries
Previews needs an executable to launch previews.  How does tihs work without an app?

1.  Edited source files
2. Target dependencies
3. Selected scheme

Previews will only select an app that's in the active scheme.

1.  Single sourcefile that's a member of an application target?  We'll use the app.
2. What if you have that sourcefile in two targets?  Here the scheme comes in, we use the app in the active scheme.
3. Say you have 2 swift files open.  We find the first common executable at the top.
4. What if I don't have an app at all?  In this case, previews makes an app on your behalf called XCPreviewAgent that will load your app.  Know the name of this process!  You'll see it in crash reports.

How to use to your advantage:
* modularize your code into libraries and create smaller schemes
* Create preview-only apps to specify needed entitlements

This lets me take advantage of smaller schemes, faster buildtimes, etc.



## Providing sample assets

Asset catalog is located here.  These images are great during development to help me test different scenarios and not require configuring every device I test with photos.  But I don't want to ship these in my app.

**Development assets**. Folders in my project, configured in build settings.  Anythingi n there is removed when I submit to the app store.

Build Settings -> Development Assets.  Manually type in the path, or i can drag the folder from the lefthand panel in there.  With that added, now this path will be removed when I submit to teh appstore.  Set up automatically by default.

One more way to provide data/images.  

## Leveraging devices

Previews lets you leverage stuff on your devices.  Simulators are fantastic.  But previews work on devices too.  maybe you want camera/sensors.  But your device has real data - photos, files, etc.  

Preview -> bottom bar -> pick device.  Bypass the simulator.  Can still configure dark mode, etc.




## Updating the navigation title while previewing on device - 25:07
```swift
.navigationTitle(“Add Collage”)
```

# Wrap up
* define previews with Preview macro for swiftui, uikit, appkit, widgets
* setup environment, sample data, and sample assets in your preview
* preview on devices to access data and capabilities
* modularize your application for faster build times and focused development

