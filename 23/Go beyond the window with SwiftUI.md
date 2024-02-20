Get ready to launch into space — a new SwiftUI scene type that can help you make great immersive experiences for visionOS. We'll show you how to create a new scene with ImmersiveSpace, place 3D content, and integrate RealityView. Explore how you can use the immersionStyle scene modifier to increase the level of immersion in an app and learn best practices for managing spaces, adding virtual hands with ARKit, adding support for SharePlay, and building an "out of this world" experience!
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

### 6:53 - RealityView in an ImmersiveSpace
```swift
ImmersiveSpace {
    RealityView { content in
        let starfield = await loadStarfield()
        content.add(starfield)
    }
}
```

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


# Resources
* 