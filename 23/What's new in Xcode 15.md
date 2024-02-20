#xcode 
Discover the latest productivity and performance improvements in Xcode 15. Explore enhancements to code completion and Xcode Previews, learn about the test navigator and test report, and find out more about the streamlined distribution process. We'll also highlight improved navigation, source control management, and debugging.

Use MAS version for production.

Last year we made watch/tv simulator optional.  This year, we're making all simulators downloadable.

Can choose which simulators to include.

# Editing
Code completion is smarter.  Now suggests name of the file!


## Build a new trait collection instance from scratch - 1:52
```swift
import Foundation
import SwiftUI
import BackyardBirdsData
import LayeredArtworkLibrary

struct PlantSummaryRow: View {
    var plant: Plant
    var body: some View {
        VStack {
            ComposedPlant(plant: plant)
                .padding(4)
                .padding(.bottom, -20)
                .clipShape(.circle)
                .background(.fill.tertiary, in: .circle)
                .padding(.horizontal, 10)
            
            VStack {
                Text(plant.speciesName)
            }
        }
    }
}
```

Default arguments are challenging.  All permutations of default arguments to pick the one you want.

More context-awareness.  We predict the best modifier based on the view type.

## Code Completion - Latitude & Longitude - 3:28
```swift
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    if let mostRecent = locations.last?.coordinate {
        logger.debug("Handled coordinate update: \(mostRecent.latitude)")
    }
}
```

we know arguments are often paired together.

color/image assets are now code-completed.  xcode 15 automatically generates symbols for each of them.

xcode 15 brinsg power of catalogs to localization experience.  Bring localization into a single place.  Edit->convert to string catalogs.

When new strings are added/removed, 

[[Discover String Catalogs]]

Documentation.

## BirdIcon Documentation - 6:18
```swift
/// Create the bird icon view.
///
/// The bird icon view is a tailored version of the ``ComposedBird`` view.
///
/// Use this initializer to display an image of a given bird.
///
/// ```swift
/// var bird: Bird
///
/// var body: some View {
///     HStack {
///         BirdIcon(bird: bird)
///             .frame(width: 60, height: 60)
///         Text(bird.speciesName)
///     }
/// }
/// ```
///
/// ![A screenshot of a view containing a bird icon with the bird's species name below it.](birdIcon)
```

Now show realtime preview of documentation. Editor->assistant -> documentation preview in the jump bar.

[[Create rich documentation with Swift-DocC]]

## macros

`@Observable`
`#Predicate`
etc.

## CaseDetection Macro - 7:37
```swift
extension TokenSyntax {
    fileprivate var initialUppercased: String {
        let name = self.text
        guard let initial = name.first else {
            return name
        }

        return "\(initial.uppercased())\(name.dropFirst())"
  }
}

public struct CaseDetectionMacro: MemberMacro {
    public static func expansion<
        Declaration: DeclGroupSyntax, Context: MacroExpansionContext
    >(
        of node: AttributeSyntax,
        providingMembersOf declaration: Declaration,
        in context: Context
    ) throws -> [DeclSyntax] {
        declaration.memberBlock.members
            .compactMap { $0.decl.as(EnumCaseDeclSyntax.self) }
            .map { $0.elements.first!.identifier }
            .map { ($0, $0.initialUppercased) }
            .map { original, uppercased in
                """
                var is\(raw: uppercased): Bool {
                    if case .\(raw: original) = self {
                        return true
                    }

                    return false
                }
                """
            }
    }
}

@main
struct EnumHelperPlugin: CompilerPlugin {
    let providingMacros: [Macro.Type] = [
        CaseDetectionMacro.self,
    ]
}
```

**cmd-shift-A: New command palette.**

expand macro with helper quick action.  Even set a breakpoint inside the macro

## Using CaseDetection Macro - 8:07
```swift
@CaseDetection
enum Element {
    case one
    case two
}

var element: Element = .one
if element.isOne {
    // Handle interesting case
}
```

[[Expand on Swift macros]]
[[Write Swift macros]]

## Previews
## New Preview API - 8:50
```swift
//Preview {
    AppDetailColumn(screen: .account)
        .backyardBirdsDataContainer()
}

//Preview("Placeholder View") {
    AppDetailColumn()
        .backyardBirdsDataContainer()
}
```

bringing preview to UIKit and AppKit!

## UIViewController Preview - 9:22
```swift
//Preview {
    let controller = DetailedMapViewController()

    controller.mapView.camera = MKMapCamera(
        lookingAtCenter: CLLocation(latitude: 37.335_690, longitude: -122.013_330).coordinate,
        fromDistance: 0,
        pitch: 0,
        heading: 0
    )
    return controller
}
```

Widget transitions

[[Build programmatic UI with Xcode Previews]]


# Navigating
bookmarks.  Right click, and select bookmark.  Shows up in navigator with a preview of its location.

xcode annotates line of code with my description.

> with a secondary click

Group bookmarks into folders.  Can be used as todo lists.  Mark when complete.  Delete altogether and choose delete bookmark.

Lines of code aren't the only thing to bookmark.  Keep track of ... todos.

bookmark any find query.
conforming types query.
Right click results and bookmark.  Query is available in the bookmark navigator.  If the results ever change, I can refresh in the right icon.

# Sharing
xc15 introduces new changes navigator and commit editor.  

Review these changes, so I'll click on uncommitted changes.  Now review all modifications.  Each section shows just enough context to understand surrounding code.  Can drag to reveal more.

Edit without leaving the view.
Status indicator highlights both staged and unstaged changes.


# Testing
Important part of shipping a high-quality app. Capture things quickly, maintain app's quality, etc.  Testing gets big improvements this year.

Updated test navigator, rewritten in Swift, when running and reporting tests in realtime.  45% faster!

Test navigator is organized around your test plan, making it easier to find tests you care about.  Use filters to find tests.

Insights analyze potentially related failures that might have been difficult to see before.  Long test runs.

**Inspect UI hierarchy.**
[[Fix failures faster with Xcode test reports]]


# Debugging
OSLog integration inXcode.




## OSLog - 17:34
```swift
import OSLog

let logger = Logger(subsystem: "BackyardBirdsData", category: "Account")

func login(password: String) -> Error? {
    var error: Error? = nil
    logger.info("Logging in user '\(username)'...")

    // ...

    if let error {
        logger.error("User '\(username)' failed to log in. Error: \(error)")
    } else {
        loggedIn = true
        logger.notice("User '\(username)' logged in successfully.")
    }
    return error
}
```

background color indicate severity.  Scan long strinsg of output into critical messages.  Choose just the fields you want to see.

When looking for something specific, can filter and narrow search.

[[Debug with structured logging]]


# Distributing

Xcode Cloud.
Great way to distribute your app.  Manages build versioning, app signing, distribution profile, etc.  This year, xcode cloud handles two more details.

1.  TestFlight test details.  New test notes, right alongside sourcecode.  Automatically attached to TF build to distribution.
2. Notarization for macOS.  All you need to do is add notarization workflow.

When your build is complete, your app is notarized and stapled.  Vital for your users, etc.  

Authors can digitally sign the contents of their frameworks.  Framework inspector has a siganture slice.  Tells you who produced and signed the framework.  Xcode remembers this identity, so you get a clear warning about the problem.

Include a PrivacyInfo manifest.  Details exactly how the framework uses sensitive data.  Also part of a signed package, and trust that its contents are signed directly from the author.


[[Verify app dependencies with digital signatures]]
[[Get started with privacy manifests]]

Xcode 15 now supports testflight internal testing distribution.  Available to your team only.  Don't release to customers.  Easy to create, just select the internal testing option when uploading to ASC.  

Xcode now bundles common distribution options.  If distributing through ASC, you get desktop notifications about build status.

[[Simplify distribution in Xcode and Xcode Cloud]]

Download today!
