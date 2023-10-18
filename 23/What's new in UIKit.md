#uikit 

# Key features
## Xcode previews

Use the preview macro to specify the name of the preview.


## Using Xcode previews with view controllers - 1:31
```swift
class LibraryViewController: UIViewController {
    // ...
}

#Preview("Library") {
    let controller = LibraryViewController()
    controller.displayCuratedContent = true
    return controller
}
```

## Using Xcode previews with views - 1:48
```swift
class SlideshowView: UIView {
    // ...
}

#Preview("Memories") {
    let view = SlideshowView()
    view.title = "Memories"
    view.subtitle = "Highlights from the past year"
    view.images = ...
    return view
}
```



## View controller lifecycle updates
`viewIsAppearing` called between `viewWillAppear` and `viewDidAppear`
* best place to update uI when view appears
* view controller and view trait collections updated
* view added to hierarchy, has accurate geometry
* back-deploys to iOS 13

1.  viewWillAppear
2. view added to hierarchy
3. view laid out by superview, traits updated
4. viewIsAppearing
5. viewWillLayoutSubviews
6. viewDidLayoutSubviews
7. transition animates
8. new transaction
9. viewDidAppear

note it is too late in viewDidAppear to make changes during the transtiion.  However viewIsAppearing is called in the same transaction as viewWillAppear.  Any changes become visible at the same time.

layout callbacks are called whenver we lay out, which may happen multiple times.  But viewIsAppearing is only called once during the appearance transition, and still gets called even if the view does not need layout.  So it's the goldilocks callback.  Not too early, not too late, not too often, just right.

## Trait system enhancements
* define custom traits
* Easily change trait values in hierarchy
* Flexible trait change registration
* UIKit and SwiftUI itneroperability

[[Unleash the UIKit trait system]]

## Animated symbol images
`.addSymbolEffect(.bounce)`
`.setSymbolImage` to transtiion
new universal animations for all symbosl
unified API across UI frameworks
composite layer annotations for custom symbols

[[Animate symbols in your app]]

## empty states
moments where there is no content to display.

## Setting up UIContentUnavailableConfiguration - 8:19
```swift
var config = UIContentUnavailableConfiguration.empty()

config.image = UIImage(systemName: "star.fill")
config.text = "No Favorites"
config.secondaryText =
    "Your favorite translations will appear here."

viewController.contentUnavailableConfiguration = config
```

here I inform that there are no favorite things.  Here I set the configuration as the `contentUnavailableConfiguration`.

UIKit also offers a loading configuration to represent content that is being repared.

## Using UIContentUnavailableConfiguration with SwiftUI - 8:56
```swift
let config = UIHostingConfiguration {
    VStack {
        ProgressView(value: progress)
        Text("Downloading file...")
            .foregroundStyle(.secondary)
    }
}
viewController.contentUnavailableConfiguration = config
```



## Using UIContentUnavailableConfiguration for search results - 9:21
```swift
override func updateContentUnavailableConfiguration(
    using state: UIContentUnavailableConfigurationState
) {
    var config: UIContentUnavailableConfiguration?
    if searchResults.isEmpty {
        config = .search()
    }
    contentUnavailableConfiguration = config
}

// Update search results for query
searchResults = backingStore.results(for: query)
setNeedsUpdateContentUnavailableConfiguration()
```



# Internationalization
Essential to deliver a consistent high-quality experience.  To facilitate that, we've made significant advancemenets in the area of font and text rendering.  In this section, I will tel lyou about
## dynamic line-height adjustments

baseline
x-height
ascenders/descenders outside these lines.
in some languages, these rqeuire significantly more vertical space.

We introduced a dynamic line height adjustment feature.  This ensures the text elemetns such as UILabels autoamtically adjust line height and vertical dimensions for optimal readability.


## Improved line-breaking and hyphenation
improved line-breaking for chinese, german, japanese, and korean
Optimized for text style and language
Adopt text styles

[[What's new with text and text interactions]]

Maybe you dispaly web content where it's different than user language.  Here you can use `typesettingLanguage` trait to adjust line-height and hyphenation rules.


## Retrieve UIImages by locale
By default, UIKit pulls image matching current variant on device.  On iOS 17, apps can request specific variants by providing a locale into image configuration.

# # Improvements for iPad
## Window dragging interaction
dragging inside any navigation bar will start moving a window.  Plays well with other GRs that might be present in your app, such as pan/swipe.

If you're not using UInavigationBar as part of the UI in your app, you can adopt `uIWindowSceneDragInteraction` and add to any view.
Can also set up gesture relationships with other pang estures in your app to ensure there areno conflicts
works with mac catalyst

## Sidebar behavior in stage manager
Sidebars are automatically hidden when necessary.  Remain hidden until requested to be shown.  When requested as narrow widths, we use overlay.  Overlay sidebar persists as window resizes larger.  When dismissed at larger width, it becomes compact style.

New behavior for column-style `UISplitViewController` only.
* tiled sidebars
* sidebars hide when necessary
* Sidebars overlaid at narrow width
* Overridwwith `preferredDispalyMode` and `preferredSplitBehavior`

## improvements for document-centric apps
New UIDocumentViewController base class
Standard appearance for document apps
UIDocument supports automatic renaming

[[Build better document-centric apps]]


## apple pencil
Hover with Apple Pencil.
Use UIHoverGestureRecognizer
zOffset is normalized distance
Render accurate ink previews with altitudeAngle and azimuthAngle
Supported by UIPointerInteraction

[[Build for the iPadOS pointer]]
Note that system pointer aren't visible when using apple pencil.  Try out hover.

New inks in pencilKit
* monoline
* fountain pen
* watercolor
* crayon

backward compatibility
* previous iOS cannot load drawings with these inks
* check requiredContentVersion on model objects
* 1 - original iOS 14.  2 - iOS 17
* Provide a message or render fallback image
* limit available inks with maximumSupportedContentVersion

## Keyboard scrolling
UIScrollView can be scrolled using keybard shortcuts
pgup, pgdown, home, end
enabled by defautl in ioS 17
override with `allowsKeyboardScrolling`


# General enhancements
## Collectionview improvements
operations with a large number of items, iOS 17 is nearly twice as fast.

layout enhancements.
in iOS 17, compositional layout gets a `NSCollectionLayoutDimension.uniformAcrossSimblings(estimate:)`
Works with self-sizing.  This ensures siblings get the largest size.

Keep in mind, this requires all sibling items to be created and sized to determine the size of the larger item. 
Use only with small numbers of sibling items.


## Spring animation parameters
* duration - perceptual.  Not fully complete.
* bounce - adds bounce to he animation, without changing how long it takes.

new method on UIView that takes these spring parameters.  So now you can say `.animate` and get a spring animation!

[[Animate with springs]]



## Text interactions
Accessories indicate input mode and dictation
resdesigned text loupe and selection handles
custom implementations can now use system-provided UI without UITextInteraction

text item actions and menus
* new api for text item interactions
* change the primary action or menu content
* tag custom ranges of text for interaction
[[What's new with text and text interactions]]

## Default status bar style

`preferredStatusBarStyle`.  
In iOS 17, the default style of status bar will change from dark to light to maintain contrast with content.  Status bar can even split styles when needed.  Since apps no longer need to specify dark/light, there are opportunities to remove customization code and make use of the default style.

## Drag and drop enhancements
Launch apps by droping supported content on to app icons.
automatically works for many document-based apps
specify `CFBundleDocumentTypes` in `Info.plist`.
Opened via `UIScene` delegate callbacks


## ISO HDR image support

supported by
* uiimageview
* uigraphicsimagerenderer
* uiimagereader
[[Support HDR images in your app]]


## Page control
Fractional page progress.  Today, page controls are commonly used to display slideshow content that page on a set duration, or along video content.

with progress and timerprogress, you can represent a fractional page progress, to provide better context on when the page will change.

automatically changes page when timer duration is met.

to sync with video as source of truth, use...
## Palette menus
palettes are a row of menu elements, used for choosing from a collection of items, ex colors.

Now it's available as a first-class control in UIKit.  Just add `.displayAsPalette` in `UIMenu` constructor.  Since elements are small, UIKit will choose the apporpriate selection indicator based on required image.  For monochromatic images, we tint.  If multicolored, a tint color stroke is drawn around.  If you are using entirely custom images or to override the builtin behaviors, you may use `selectedImage` property.

# Next steps
* compile your app for iOS 17
* adopt new UIKit APIs
* Verify your app's layout for a variety of languages
* 