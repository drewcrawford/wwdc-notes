#docc 

Together with my collage ethan, I'm excited to give you a tour of swift-docc.  Create even better documentation. 

Last year, allowing you to guide developers through your swift frameworks.
* reference
* articles
* tutorials

# New in Swift-DocC
* app projects
* Objective-C and C APIs
* Publish to managed hosting services
* navigation sidebar

# Document
Good documentation is essential to any software project.  As your project evolves, it's important to document design and implementation.

This year, we're excited to expand to app projects.

Product => Build documentation.  Stubs that we automatically generated.  Great starting point for filling in these pages to help contributors navigate the proejct.

Teach how each API works individually.  
```swift
/// A view that displays a sloth.
///
/// This is the main view of ``SlothyApp``.
/// Create a sloth view by providing a binding to a sloth.
///
/// ```swift
/// @State private var sloth: Sloth
///
/// var body: some View {
///     SlothView(sloth: $sloth)
/// }
/// ```
struct SlothView: View {
    // ...
}
```

Start your comment using `///`.  add a concise summary.  in build documentation, this translates into text below view's name.

Add more detail.  This appears in overview.

Link syntax.  Active links, allowing you to quickly jump to pages to learn more.  Validates these links when you build, so you get warnings if you go out of date.

Add a code listing using markdown code syntax.  Now contributors know at a glance how to use this view.

```swift
struct SlothView: View {
    /// Creates a view that displays the specified sloth.
    ///
    /// - Parameter sloth: The sloth the user will edit.
    init(sloth: Binding<Sloth>) {
				// ...
    }
}
```

for initializers and methods, it's a good idea to document each parameter individually.  Notice how the content appears in a separate parameter section.

ObjC API.  Excited to bring this to objc.

* same documentation format
* source editor integration
* Mixed Swift + ObjC apps and frameworks

```objc
/// A sound that can be played.
///
/// Use an instance of this type to play a sound
/// to the user in response to an action or
/// event.
@interface SLOSound : NSObject

/// Creates a sound given its name and path on
/// disk.
///
/// - Parameters:
///   - name: The name of the sound.
///   - filePath: The path to the sound file on disk.
- (id)initWithName:(NSString *)name
          filePath:(NSString *)filePath;

@end
```

Because this class can be used from both swift and objc code, xcode displays a language toggle in built documentation.  

Let's focus on top-level page.  Great introduction to what the app does.  

1.  Add a documentation catalog.  Right click on project source files, and pick new file.  Documentation catalog

Complements your documentation wilth additional media.

```markdown
# ``Slothy``

An app to create and care for custom sloths.

## Overview

Slothy is an iOS app that allows users to create and care for virtual sloths.

![An illustration displaying the UI for finding, creating, and taking care of a sloth in Slothy.](slothy.png)

The Slothy app project contains views to present Slothy's user interface, and utilities to play sounds as the user interacts with the app.
```

This looks much more inviting.  Now contributors know at a glance what this is about.

Now that we've seen how to write and build documentation, time to publish.

# Publish
We've been developing the app in a modular way.  Alongside a more generally useful package called SlothCreator.  Publish to a wider audience so that other developers can make use of it.

Bundle is called a DocC archive, a portable container for your documentation.  Export directly from xcode's documentation window.  Not just a portable contianer for opeining in xcode.  It contains a fully-featured website.  new in xcode 14, that archive is directly compatible with most webservers.  Publishing your docs are easier than ever.  *Deploy your documentation by copying into the root of the webserver*.  Compatible with most managed services, including Github Pages.

It makes sense for us to publish our docs there as well.  Unlike a standard server, your website is not published at the root path of the URL, but at a base path.  You need to specify this with a build setting.  To fully understand this, and why we only need in certain circumstances...

Suppose we have https://slothy.example.com.  And we publish into that website.
If we take content of the archive to https://slothy.example.com/documentation/slothcreator.  
Any tutorials for the sloth creator package is in tutorials.

But in this case, we're not publishing to our own domain, to keep with github repository, we publish to github pages.  When you create a pages site for a repository, it's at some base path corresponding to the name of the repository.  Generally that is somethign like https://username.github.com/repository

Any reference and tutorial documentation paths are appended to the base path.  Since unique to your repository, tell docc before you build.

*DocC Archive Hosting Base Path* setting.  

## Demo

## Automating deployments
* Swift-DocC plugin
* Use this plugin to simplify the process of building documentation for swift packages.
* `xcodebuild docbuild`
	* docs linked below


# Browse

All-new browsing experience.  Navigation sidebar.

Now i can see pages that are organized as children, without needing to open the page.

The state of the navigation sidebar stays constant, allowing me to keep track of the pages I've already visited.  Natural exploration of the framework.

What if I'm interested in a specific symbol? Filter field.  energyLevel.  Documentation is exactly what I was looking for.

New browsing experience will bring your documentation sites to the next leve.

# Wrap up
* support for documenting all of your projects
* Compatible with popular hosting services
* Powerful new navigation experience

[[Improve the discoverability of your Swift-DocC content]]
[[Build interactive tutorials using DocC]]

https://developer.apple.com/documentation/Xcode/documenting-apps-frameworks-and-packages
https://apple.github.io/swift-docc-plugin/documentation/swiftdoccplugin/
https://developer.apple.com/documentation/DocC
https://developer.apple.com/documentation/xcode/slothcreator_building_docc_documentation_in_xcode
