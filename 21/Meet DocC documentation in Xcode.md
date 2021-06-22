Introduction to new documentation features in Xcode 13

Unlock new possiblities.  Xcode 13 has new features to build, write, and browse documentation.

Read your swift dependencies while you write your code.

Lives alongside platform docs.

# Overview
What comes in a package?  XC13 has a compiler for documentation.

Build/view docs for frameworks / packages.  DocC.

Enhance the way you read/write documentation.

More than a documentation compiler.  It's an integrated doc environment.  Rich live environment with quickhelp, code completion, Full documentation, etc.

Stay tuned for other docc sessions.

Write excellent reference documentation.  Complete with the ability to describe how APIs are working together.

Additional ways to write doc with docc.

* Articles – allow you to walk users through the big picture behind your framework
* Tutorials – powerful step-by-step walkthrough of writing a project

[[Elevate your DocC documentation in Xcode]]
[[Build interactive tutorials using DocC]]

Open source
*Later this year*

Webapp?


# Building and browsing
How it works?

Xcode builds your framework or package and asks the compiler to save information about public APIs alongside artifacts.

Public API info is handed to DocC.  Which combines it with your documentation catalog (articles, tutorials) written outside your code.

[[Elevate your DocC documentation in Xcode]]

To learn more, check out the session.

Thanks to DocC integratino, this process repeats for every swift package and framework you depend on.

What does this mean for day-to-day needs?  3 ways to build documentation.

1.  Product=>Build Documentation item.
2.  Build documentation during build => Yes (build setting)
3.  `xcodebuild docbuild`

[[Host and automate your DocC documentation]]

## Demo
Jump bar

# Authoring
In-source documentation.

To add documentation, write a special comment directly above.  Fits in with your existing text-based tooling.

```swift
// A model representing a sloth.
public struct Sloth {
    // ...
}
```

What about adopters?  Doc comments.  By writing a comment that begins with `///`, doc compiler will associate that with declaration.

`/**`.  

Swift package that provides functionality for cataloging and customizing sloths.  I want to be sure that there's an easy way to ensure adopters have a way to get started.  

In particular, I want to be sure that every public API is well-documented because these APIs are accessible to anyone importing.  

DocC only generates doc pages for public and open symbols.  

```swift
/// Food that a sloth can consume.
///
/// Sloths love to eat the leaves and twigs they find in the rainforest canopy as they
/// slowly move around. To feed them these items, you can use the `twig`,
/// `regularLeaf` and `largeLeaf` default foods.
///
/// ```swift
/// superSloth.eat(.twig)
/// ```
public struct Food {
		// ...
}
```

Line break.  Discussion section.

Add a code example using markdown codeblocks.

QuickHelp.  Hold down option.  "Open in developer documentation".

"Open in Developer Documentation" link.

```swift
/// A model representing a sloth.
public struct Sloth {
    /// Sleep in the specified habitat for a number of hours.
    ///
    /// - Parameters:
    ///     - habitat: The location for the sloth to sleep.
    ///     - numberOfHours: The number of hours for the sloth to sleep.
    /// - Returns: The sloth’s energy level after sleeping.
    mutating public func sleep(in habitat: Habitat, for numberOfHours: Int = 12) -> Int {
        energyLevel += habitat.comfortLevel * numberOfHours
        return energyLevel
    }
}
```

I could manually write doc comment.  But for a more complicated symbol, how to add documentation?

Cmd and click on declaration.  Then use "Add Documentation" action which creates a template.

Rebuild documentation with Product=>Build Documentation.  Once again, Xcode is bilding documentation alongside framework.

Reading this page of documentation, I don't have great context of relevant symbols.  How to call out symbols that my reader should consider learning about?

**Link to symbols!!!**

* Make connections
* Encourage discovery
* Code completion


Reader should understand side effects, etc.

```swift
/// A model representing a sloth.
public struct Sloth {
    /// The energy level of the sloth.
    public var energyLevel: EnergyLevel

    /// Sleep in the specified habitat for a number of hours.
    ///
    /// Each time the sloth sleeps, their ``energyLevel`` increases every hour by the
    /// habitat's ``Habitat/comfortLevel``.  
    ///
    /// - Parameters:
    ///     - habitat: The location for the sloth to sleep.
    ///     - numberOfHours: The number of hours for the sloth to sleep.
    /// - Returns: The sloth’s energy level after sleeping.
    mutating public func sleep(in habitat: Habitat, for numberOfHours: Int = 12) -> Int {
        energyLevel += habitat.comfortLevel * numberOfHours
        return energyLevel
    }
}
```

Before xc13, the natural way was just to write in 1 backtick.  But now I can transform to double-backticks and create a link.

Now that was simple, because it's a sibling.  So I didn't have to qualify it.  But a child of a different type requires more specificity.  `/` links to child.

Code completion works in doc linking.  These links are accessible in QuickHelp.

In the discussion are two links.  If I click on one, I'm brought to reference symbols page.

# Sharing

* Documentation archives
* Open directly in xcode documentation window
* publish to the web

[[Host and automate your DocC documentation]]

> single-page webapp

Can export/import directly from documentation window.

Double-click on doc archive and xcode opens.  

# Wrap up
* Powered by DocC
* Document directly in-source
* Fully integrated with XC13

[[Elevate your DocC documentation in Xcode]]
[[Host and automate your DocC documentation]]
[[Build interactive tutorials using DocC]]




