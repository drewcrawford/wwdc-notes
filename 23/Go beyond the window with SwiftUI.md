Get ready to launch into space — a new SwiftUI scene type that can help you make great immersive experiences for visionOS. We'll show you how to create a new scene with ImmersiveSpace, place 3D content, and integrate RealityView. Explore how you can use the immersionStyle scene modifier to increase the level of immersion in an app and learn best practices for managing spaces, adding virtual hands with ARKit, adding support for SharePlay, and building an "out of this world" experience!

ARKit.

Immersive experiences.  In these experiences, your application displays UI, including windows and three-dimensional content anywhere around you.  Surroundings remain visible, and become part of the experience.

Anchor elements of your app to surfaces and augment the real world, etc.


[[Take SwiftUI to the next dimension]]

This year we've added the third dimension to swiftUI.  Windows and volumes on xrOS and display 3d ui elements with easy to use declarative patterns of swiftui.  Both windows and volumes let you display content within their bounds.

How to create a truly immersive experience?  Beyodn the window's bounds, all around your head, etc.  

Spaces are a kind of container to present your user interface on xrOS

# Get started with Space

### 4:18 - Defining an ImmersiveSpace
```swift
@main
struct WorldApp: App {
    var body: some Scene {
        ImmersiveSpace {
            SolarSystem()
        }
    }
}
```

Only one space open at a time.

Place view hierarchy in body of scene.  By placing this in immersivespace, rendered without clipping boundaries.

When you display immersive space, you enter the full space.  All other apps will go away.  To make room for your content to appear.



# Display content


### 6:53 - RealityView in an ImmersiveSpace
```swift
ImmersiveSpace {
    RealityView { content in
        let starfield = await loadStarfield()
        content.add(starfield)
    }
}
```

We encourage you to use ImmersiveSpace together with RealityView.  Immersive space and RealityView work hand in hand, and provide all features you need for great immersive experiences.

ex, builtin support for asynchronous loading of assets.  

* asynchronous loading
* ARKit anchor-based placement
* Use of hands and head-pose data

In SwiftUI, y-axis is downwards, z-axis is towards you.  
In RK, y-axis points upwards.

[[Build spatial experiences with RealityKit]]

### 8:17 - ImmersiveSpace with a SolarSystem view
```swift
@main
struct WorldApp: App {
    var body: some Scene {
        ImmersiveSpace(id: "solar") {
            SolarSystem()
        }
    }
}
```

we use id later to open the space.

### 9:00 - LaunchWindow
```swift
struct LaunchWindow: Scene {
    var body: some Scene {
        WindowGroup {
            VStack {
                Text("The Solar System")
                    .font(.largeTitle)
                Text("Every 365.25 days, the planet and its satellites [...]")
                SpaceControl()
            }
        }
    }
}
```

### 9:11 - SpaceControl button using Environment actions for opening and dismissing an ImmersiveSpace scene
```swift
struct SpaceControl: View {
    @Environment(\.openImmersiveSpace) private var openImmersiveSpace
    @Environment(\.dismissImmersiveSpace) private var dismissImmersiveSpace
    @State private var isSpaceHidden: Bool = true
    var body: some View {
        Button(isSpaceHidden ? "View Outer Space" : "Exit the solar system") {
            Task {
                if isSpaceHidden {
                    let result = await openImmersiveSpace(id: "solar")
                    switch result {
                        // Handle result
                    }
                } else {
                    await dismissImmersiveSpace()
                    isSpaceHidden = true
                }
            }
        }
    }
}
```

Use these actions when button is invoked.  When opening the space, we pass in id from earlier.  Only one space can be open at a time, dismiss doesn't need any argument.

System animates in and out with a certain duration.  These actions are async so that you can react.

Opening may fail, it tells you whether it failed/succeeded.


### 10:44 - WorldApp using LaunchWindow and ImmersiveSpace
```swift
@main
struct WorldApp: App {
    var body: some Scene {
        LaunchWindow()
        ImmersiveSpace(id: "solar") {
            SolarSystem()
        }
    }
}
```

Keep in mind, it may take time for your assets to be fully loaded and ready to be rendered.  Make sure to leverage new Model3D and RealityView APIs which load your 3d assets asynchronously.

### 11:32 - Model3D with phase handling
```swift
Model3D(named: "Earth") { phase in
    switch phase {
        case .empty:
            Text( "Waiting" )
        case .failure(let error):
            Text("Error \(error.localizedDescription)")
        case .success(let model):
            model.resizable()
    }
}
```

# Managing your Space
* scene phases
* coordinate conversions
* immersion styles

we may leave the space involuntarily, e.g. alerts.  "inactive phase".



### 13:04 - Scene Phases
```swift
@main
struct WorldApp: App {
    @EnvironmentObject private var model: ViewModel
    @Environment(\.scenePhase) private var scenePhase

    ImmersiveSpace(id: "solar") {
        SolarSystem()
            .onChange(of: scenePhase) {
                switch scenePhase {
                case .inactive, .background:
                    model.solarEarth.scale = 0.5
                case .active:
                    model.solarEarth.scale = 1
                }
            }
    }
}
```

Makes handling these transitions easy and convenient!

Coordinate conversion.  To position model next to window, need to know window position in coordinate system.

### 14:21 - Coordinate Conversions
```swift
var body: some View {
    GeometryReader3D { proxy in
        ZStack {
            Earth(
                earthConfiguration: model.solarEarth,
                satelliteConfiguration: [model.solarSatellite],
                moonConfiguration: model.solarMoon,
                showSun: true,
                sunAngle: model.solarSunAngle,
                animateUpdates: animateUpdates
            )
            .onTapGesture {
                if let translation = proxy.transform(in: .immersiveSpace)?.translation {
                    model.solarEarth.position = Point3D(translation)
                }
            }
        }
    }
}
```

Using this transform we do the conversion!

[[Build spatial SharePlay experiences]]

## Immersion styles

* Mixed
* Progressive
* Full



### 16:34 - Immersion Styles
```swift
@main
struct WorldApp: App {
   @State private var currentStyle: ImmersionStyle = .mixed
   var body: some Scene {
        ImmersiveSpace(id: "solar") {
            SolarSystem()
                .simultaneousGesture(MagnifyGesture()
                    .onChanged { value in
                        let scale = value.magnification
                        if scale > 5 {
                            currentStyle = .progressive
                        } else if scale > 10 {
                            currentStyle = .full
                        } else {
                            currentStyle = .mixed
                        }
                    }
                )
        }
        .immersionStyle(selection:$currentStyle, in: .mixed, .progressive, .full)
   }
}
```

progressive -> controlled by digital crown?



# Customization


Our app allows us to open a space with a button, but what about on launch?

In order to launch directly to an immersive space, need to configure scene manifest.  

Surrounding effects, space passthrough.  Surroundings can be dimmed.  We set the `preferredSurroundingEffects` to be dark, so our surroundings are automatically dimmed.  Since no passthrough is available.

### 20:08 - Surrounding Effects
```swift
@main
struct WorldApp: App {
  @State private var currentStyle: ImmersionStyle = .progressive
    var body: some Scene {
        ImmersiveSpace(id: "solar") {
            SolarSystem()
                .preferredSurroundingsEffect( .systemDark)
        }
        .immersionStyle(selection: $currentStyle, in: .progressive)
     }
}
```

### 20:30 - Upper Limbs Visibility
```swift
@main
struct WorldApp: App {
    @State private var currentStyle: ImmersionStyle = .full
    var body: some Scene {
        ImmersiveSpace(id: "solar") {
            SolarSystem()
        }
        .immersionStyle(selection: $currentStyle, in: .full)
        .upperLimbVisibility(.hidden)
    }
}
```

can show virtual hands instead.

### 20:52 - Hand Anchoring
```swift
struct SpaceGloves2: View {

    let arSession = ARKitSession()
    let handTracking = HandTrackingProvider()

    var body: some View {

        RealityView { content in

            let root = Entity()
            content.add(root)

            // Load Left glove
            let leftGlove = try! Entity.loadModel(named: "assets/gloves/LeftGlove_v001.usdz")
            root.addChild(leftGlove)

            // Load Right glove
            let rightGlove = try! Entity.loadModel(named: "assets/gloves/RightGlove_v001.usdz")
            root.addChild(rightGlove)

            // Start ARKit session and fetch anchorUpdates
            Task {
                do {
                    try await arSession.run([handTracking])
                } catch let error as ProviderError {
                    print("Encountered an error while running providers: \(error.localizedDescription)")
                } catch let error {
                    print("Encountered an unexpected error: \(error.localizedDescription)")
                }
                for await anchorUpdate in handTracking.anchorUpdates {
                    let anchor = anchorUpdate.anchor
                    switch anchor.chirality {
                    case .left:
                        if let leftGlove = Entity.leftHand {
                            leftGlove.transform = Transform(matrix: anchor.transform)
                            for (index, jointName) in anchor.skeleton.definition.jointNames.enumerated() {
                                leftGlove.jointTransforms[index].rotation = simd_quatf(anchor.skeleton.joint(named: jointName).localTransform)
                            }
                        }
                    case .right:
                        if let rightGlove = Entity.rightHand {
                            rightGlove.transform = Transform(matrix: anchor.transform)
                            for (index, jointName) in anchor.skeleton.definition.jointNames.enumerated() {
                                rightGlove.jointTransforms[index].rotation = simd_quatf(anchor.skeleton.joint(named: jointName).localTransform)

                            }
                        }
                    }
                }
            }
        }
    }
}
```

For more details

[[Evolve your ARKit app for spatial experiences]]


* create immersive experiences effortlessly
* style your space with unique presentations
* multiple options for customization!


# Resources
* 