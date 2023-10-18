Find out how you can build immersive apps for visionOS using Xcode and Reality Composer Pro. We'll show you how to get started with a new visionOS project, use Xcode Previews for your SwiftUI development, and take advantage of RealityKit and RealityView to render 3D content.

# Create an Xcode project
Note that you may need to download platform support if not installed.

specify the type of initial scene that's already included.  Asa  developer, can add additional scenes later.  Same type, or different scene types.

template offers two initial... window and volume.

## Windows
primarily two-dimensional.  Resized, but depth is fixed.  Windows a regenerally shown alongside other running apps.

[[Meet SwiftuI for spatial computing]]
## Volumes
Primarily 3D content.  Sizes in all 3 dimensions are controlled by the app, but cannot be adjusted by user.

Shown alongside other running apps.

[[Take SwiftUI to the next dimension]]

## Immersive space

Present unbounded content anywhere on the infinite canvas.  Other running apps are hidden.  App can also access dedicated rendering resources, and request permission to enable ARKit features like hand tracking.

Mixed immersion -> like AR, passthrough
progressive immersion -> portal, doesn't remove peopel from their surroundings.  Roughly 180 degree view, digital crown to adjust size
full immersion - hides passthrough entirely

[[Go beyond the window with SwiftUI]]

By default, no immersive space is added.  

Provide clear entry/exit controls, so people can be more immersed in your content.  Avoid shoving people into immersion.

Most of the code in the new project is in content view.  Uses new platform-specific features.

`ContentView`.  Defines a single `@State var enlarge` that's used for a simple effect.  

Content provided by `body`.  Two views, nested in a VStack.  Makes the nested views stakc ok.

`RealityView` -> new to this platform.

Standard swiftUI toggle view.  Toggles the enlarge property.

Good chance you've already seen the toggle view.  Most SwiftUI controls that are supported on other platforms work as expected.

Two closures as parameters, a make and update closure.
* make - initial content to the view.  If it succeeds, it adds the loaded content to the view using `content.add`.  Can also generate procedurally, etc.
* update - optional, called whenever the swiftui state changes.  Gets first entity, etc.
###  Glass background effect - 6:54
```swift
VStack {
        Toggle("Enlarge RealityView Content", isOn: $enlarge)
            .toggleStyle(.button)
    }
    .padding()
    .glassBackgroundEffect()
```

###  RealityView - 7:28
```swift
RealityView { content in
    // Add the initial RealityKit content
    if let scene = try? await Entity(named: "Scene", in: realityKitContentBundle) {
        content.add(scene)
    }
} update: { content in
    // Update the RealityKit content when SwiftUI state changes
    if let scene = content.entities.first {
        let uniformScale: Float = enlarge ? 1.4 : 1.0
        scene.transform.scale = [uniformScale, uniformScale, uniformScale]
    }
}
.gesture(TapGesture().targetedToAnyEntity().onEnded { _ in
    enlarge.toggle()
})
```

update is not called on every frame, just when swiftui state changes.

[[Build spatial experiences with RealityKit]]

# Simulator

bottom right corner - control the simulator device.  Allows you to look around, pan, orbit, and move forward/backwards.  

switch between interacting with content and moving around.  

For example, if I click ont he pan button, then clicking and draggintg the viewport pans the view.

clicking the leftmost control switches back.

Several simulated themes to see your pap running in different rooms and lighting conditions.  

For more information, see the documentation.  Let's take a look at our new app running there.

Tap gesture updates the swiftui state to cause both things to react to state changes.
# Xcode previews
focus/iterate on look and behavior of your app's views.

as with the simulator, xcode previews are presented as a simulated device view.  use the same controls to navigate the preview window.


# Reality composer pro
New tool to help you work with realitykit content.

Great place to prepar and preview spatial content for your apps.

App's content view uses realitykit view.

Located in packages group in the xcode project.

realitykitcontent packages are swift packages containing realitykit content.  Processed at buildtime.  

we see the contents of the content package.  

while xcode's main focus is on app stuff, reality composer pro is good for editing RK content.

organizes content into scenes.  Content package starts with a single scene.  In order to enhance our project, let's create a new scene to create content for an immersive space.  New->scene.  Give it a name, in this case it's just the immersive scene.  Then click save.

Use the ineferred position of your feet as the origin.
+x -> to your right
+y -> up
-z axis -> in front of you

full space apps can request additional data, such as hand orientation.  Some data is privacy-sensitive.  Person using the app will be prompted.

Not available to apps in the sahred space

[[Meet ARKit for spatial computing]]

My usdz cloud model will be used to create content for an immersive experience.

double-click in scene hierarchy to focus.

[[Meet Reality Composer Pro]]
[[Work with Reality Composer Pro content in Xcode]]

# Create an immersive scene
###  ImmersiveView - 20:31
```swift
// MyFirstImmersiveApp.swift

@main
struct MyFirstImmersiveApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }.windowStyle(.volumetric)

        ImmersiveSpace(id: "ImmersiveSpace") {
            ImmersiveView()
        }
    }
}
```

how does our app know to present `ContentView`?  See snippet above.

Default content for immersive view is just a text box.  

By default, previews are clipped to default scene bounds.  If there's content out of bounds, it won't be visible.  To preview immersive content, modify the view being prepared to `sizeThatFits`
###  Size that fits - 22:58
```swift
#Preview {
    ImmersiveView()
        .previewLayout(.sizeThatFits)
}
```

How to open additional (immersive) scenes

###  openImmersiveSpace - 23:48
```swift
struct ContentView: View {

    @Environment(\.openImmersiveSpace) var openImmersiveSpace

    var body: some View {
        Button("Open") {
            Task {
                await openImmersiveSpace(id: "ImmersiveSpace")
            }
        }
    }
}
```

Experience your content in the simulator, but immersion is particularly compelling from the device itself.

While a person can move the app's volume anywhere, the immersive space is placed in a fixed position.  You move yourself around inside the immersive space.
# Target gestures to entities


Imagine that tapping on a cloud causes it to float gently.

attach gestures to entities.

Note that realityview may have content with multiple entities.  If a person taps on one of the clouds, swiftUI invokes the gesture on the realityview.  How do we know which cloud is targeted?

`targetedToAnyEntity` determines on the exact entity that we targeted.  Other ways of targeting entities.  Specific entity, or all entities matching a query.  See documentation.


###  Entity targeting - 25:48
```swift
import SwiftUI
import RealityKit

struct ContentView: View {
    var body: some View {
        RealityView { content in
            // For entity targeting to work, entities must have a CollisionComponent
            // and an InputTargetComponent!
        }
        .gesture(TapGesture().targetedToAnyEntity().onEnded { value in
            print("Tapped entity \(value.entity)!")
        })
    }
}
```

when the tap occurs, we'll add an animation.

IN RC pro, select both clouds at once.  Then click the 'add component' button at the bottom of the inspector panel, and select collision.  in the inspector panel, we see that the collision component has been added to the clouds.  Automatically choose an improtant disclosure shape.

We now do the same for the input targets.  Click the 'add componen't button again, this time selecting input target.

File->save all.

Let's use an RK animation in the gesture handler.
###  Move animation - 28:56
```swift
.gesture(TapGesture().targetedToAnyEntity().onEnded { value in
    var transform = value.entity.transform
    transform.translation += SIMD3(0.1, 0, -0.1)
    value.entity.move(
        to: transform,
        relativeTo: nil,
        duration: 3,
        timingFunction: .easeInOut
    )
})
```

Entity targeting is the glue that connects swiftui interactions to RK content.  In am ore complex app, you can use entity targeting to trigger more sophisticated actions such as presenting views, audio, animations, etc

# Wrap up
* create an xcode proejct
* simulator and xcode previews
* reality composer pro
* create an immersive scene
* entity targeting
