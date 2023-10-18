Learn how you can take advantage of the latest features in Swift-DocC to create rich and detailed documentation for your app or framework. We'll show you how to use the Xcode 15 Documentation Preview editor to efficiently iterate on your existing project's documentation, and explore expanded authoring capabilities like grid-based layouts, video support, and custom themes. To get the most out of this session, you should have a working knowledge of the basics of Swift-DocC documentation.

brand new documentation preview editor.  

# Document in-source
# Simplified Publishing

# Fully integrated

Now been open-sourced.

# Documentation preview editor
Real-time view of rendered version of your documentation.

* grid-based layouts
* etc.
Can craft truly bespoke documentation that really advocates for your pdouct.

See earlier talks for basic info [[Meet DocC documentation in Xcode]] [[Elevate your DocC documentation in Xcode]]

# Writing

Begin by evaluating the current stat eof its documentation.

Product -> Build Documentation

Xcode will also produce documentation for the dependencies.  So if you depend on 3rd-party libraries, these appear as well.

Topics are used to organize different pages into logical groups.  First we have essentials, which contains introductory documentation.

Then symbols for creation, etc., each topic.

* Learn how to best organize your  documentation into logical topic groups
* [[Improve the discoverability of your Swift-DocC content]]

New feature: *extension support*.  You can now document extensions you make to external types.  Driven by community contributions.

Can open the assistant editor to 'documentation preview'.  

* Supported in Swift Packages and Xcode projects
* Include additional documentation content

```swift
import SwiftUI

/// An extension that facilitates the display of sloths in user interfaces.
public extension Image {
    /// Create an image from the given sloth.
    ///
    /// Use this initializer to display an image representation of a
    /// given sloth.
    ///
    /// ```swift
    /// let iceSloth = Sloth(name: "Super Sloth", color: .blue, power: .ice)
    ///
    /// var body: some View {
    ///     Image(iceSloth)
    ///         .resizable()
    ///         .aspectRatio(contentMode: .fit)
    ///     Text(iceSloth.name)
    /// }
    /// ```
    ///
    /// ![A screenshot of an ice sloth, with the text Super Sloth underneath.](iceSloth)
    ///
    /// This initializer is useful for displaying static sloth images.
    /// To create an interactive view containing a sloth, use ``SlothView``.
    init(_ sloth: Sloth) {
        self.init("\(sloth.power)-sloth")
    }
}
```

## Directives
 * extend markdown syntax
 * attach extra metadata to page
 * create custom layouts

Keep in midn that these directives are intended to be used for later.  
problems:
difficult to discover link
Images are not clearly associated wtih their paragraphs.
Different versions of the same image.
styled as regular article

Define rows and columns
```swift
@Row {
    @Column(size: 2) {
        First, you customize your sloth by picking its 
        ``Sloth/power-swift.property``. The power of your sloth influences
        its abilities and how well they cope in their environment. The app
        displays a picker view that showcases the available powers and
        previews your sloth for the selected power.
    }
    
    @Column {
        ![A screenshot of the power picker user interface with four powers displayed – ice, fire, wind, and lightning](slothy-powerPicker)
    }
}

@Row {
    @Column {
        ![A screenshot of the sloth status user interface that indicates the the amount of sleep, fun, and exercise a given sloth is in need of.](slothy-status)
    }
    
    @Column(size: 2) {
        Once you've customized your sloth, it's ready to ready to thrive.
        You'll find that sloths will happily munch on a leaf, but may not be as 
        receptive to working out. Use the activity picker to send some
        encouragement.
    }
}
```

Tab navigator allowing you to collapse multiple elements into one.

```swift
@TabNavigator {
    @Tab("English") {
        ![Two screenshots showing the Slothy app rendering with English language content. The first screenshot shows a sloth map and the second screenshot shows a sloth power picker.](slothy-localization_eng)
    }
    
    @Tab("Chinese") {
        ![Two screenshots showing the Slothy app rendering with Chinese language content. The first screenshot shows a sloth map and the second screenshot shows a sloth power picker.](slothy-localization_zh)
    }
    
    @Tab("Spanish") {
        ![Two screenshots showing the Slothy app rendering with Spanish language content. The first screenshot shows a sloth map and the second screenshot shows a sloth power picker.](slothy-localization_es)
    }
}
```

Click through each tab to access interesting screenshots.

```swift
@Video(poster: "slothy-hero-poster", source: "slothy-hero", alt: "An animated video showing two screens in the Slothy app. The first screenshot shows a sloth map and the second screenshot shows a sloth power picker.")
```

```swift
@Metadata {
    @CallToAction(purpose: link, url: "https://example.com/slothy-repository")
}
```

```swift
@Metadata {
    @CallToAction(purpose: link, url: "https://example.com/slothy-repository")
    @PageKind(sampleCode)
}
```

consider telling us about other page kinds that might be interesting to you.

```swift
@Links(visualStyle: detailedGrid) {
    - <doc:GettingStarted>
    - <doc:SlothySample>
}
```

metadata can have page images

```swift
@Metadata {
    @PageImage(
        purpose: card, 
        source: "slothy-card", 
        alt: "Two screenshots showing the Slothy app. The first screenshot shows a sloth map and the second screenshot shows a sloth power picker.")
}
```

```swift
@Metadata {
    @PageImage(
        purpose: icon, 
        source: "slothCreator-icon", 
        alt: "A technology icon representing the SlothCreator framework.")
}
```

```swift
@Metadata {
    @PageColor(green)
}
```

# Theming

Part of a larger product website.  Ensure this is visually consistent with the product site.

* Supports highly-detailed customizations
* authored in JSON
* integrate wtih the rest of your website

```json
{
    "theme": {
        "color": {
            "standard-green": "#83ac38"
        },
        "typography": {
            "html-font": "serif"
        }
    }
}
```

* themes are site-wide, not-page-specific, use directives for per-page info
* deployment-specific.  Don't change the xcode
* Consider when to use theming and when to use a metadata directive (directives work in xcode)





# Navigation

Brand new quick navigation feature.  
Smilar to open quickly
activate keyboard shortcut, etc.
shift-cmd-O

# Wrap up
* try the nwe documentation preview editor
* Enrich your documentation with layout directives
* Consider adding a custom theme (web only)



# Resources



* https://developer.apple.com/documentation/docc

[[What's new in Xcode 15]]

[[Improve the discoverability of your Swift-DocC content]]
[[What's new in Swift-DocC]]
[[Build interactive tutorials using DocC]]
[[Elevate your DocC documentation in Xcode]]
[[Meet DocC documentation in Xcode]]
