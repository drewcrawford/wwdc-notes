Take a tour of the solar system with us and explore SwiftUI for visionOS! Discover how you can build an entirely new universe of apps with windows, volumes, and spaces. We'll show you how to get started with SwiftUI on this platform as we build an astronomy app, add 3D content, and create a fully immersive experience to transport people to the stars.

### 2:02 - Button
```swift
Button("Of course") {
  // perform action
}
```

### 2:41 - Toggle Favorite
```swift
Toggle(isOn: $favorite) {
    Label("Favorite", systemImage: "star")
}
```

### 2:48 - TabView
```swift
TabView {
    DogsTab()
        .tabItem {
            Label("Dogs", systemImage: "pawprint")
        }

    CatsTab()
        .tabItem {
            Label("Cats", image: "cat")
        }

    BirdsTab()
        .tabItem {
            Label("Birds", systemImage: "bird")
        }
}
```

### 3:37 - World App
```swift
@main
struct WorldApp: App {
    var body: some Scene {
        WindowGroup("Hello, world") {
            ContentView()
        }
    }
}
```

### 7:03 - World TabView
```swift
@main
struct WorldApp: App {
    var body: some Scene {
        WindowGroup("Hello, world") {
            TabView {
                Modules()
                    .tag(Tabs.menu)
                    .tabItem {
                        Label("Experience", systemImage: "globe.americas")
                    }
                FunFactsTab()
              	    .tag(Tabs.library)
                    .tabItem {
                        Label("Library", systemImage: "book")
                    }                    
            }
        }
    }
}
```

### 8:42 - Stats Grid Section
```swift
VStack(alignment: .leading, spacing: 12) {
    Text("Stats")
        .font(.title)

    StatsGrid(stats: stats)
        .padding()
        .background(.regularMaterial, in: .rect(cornerRadius: 12))
}
```

### 9:23 - Fun Fact Button
```swift
Button(action: {
    // perform button action
}) {
    VStack(alignment: .leading, spacing: 12) {
        Text(fact.title)
            .font(.title2)
            .lineLimit(2)
        Text(fact.details)
            .font(.body)
            .lineLimit(4)
        Text("Learn more")
            .font(.caption)
            .foregroundStyle(.secondary)
    }
    .frame(width: 180, alignment: .leading)
}
.buttonStyle(.funFact)
```

### 13:15 - FunFactButtonStyle
```swift
struct FunFactButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding()
            .background(.regularMaterial, in: .rect(cornerRadius: 12))
            .hoverEffect()
            .scaleEffect(configuration.isPressed ? 0.95 : 1)
    }
}
```

### 14:17 - Globe Volume
```swift
@main
struct WorldApp: App {
    var body: some Scene {
        WindowGroup {
            Globe()
        }
        .windowStyle(.volumetric)
        .defaultSize(width: 600, height: 600, depth: 600)
    }
}
```

### 14:36 - Model3D
```swift
import SwiftUI
import RealityKit

struct Globe: View {
    var body: some View {
        Model3D(named: "Earth")
    }
}
```

### 15:40 - Globe with rotation and controls
```swift
struct Globe: View {
    @State var rotation = Angle.zero
    var body: some View {
        ZStack(alignment: .bottom) {
            Model3D(named: "Earth")
                .rotation3DEffect(rotation, axis: .y)
                .onTapGesture {
                    withAnimation(.bouncy) {
                        rotation.degrees += randomRotation()
                    }
                }
                .padding3D(.front, 200)
            
            GlobeControls()
                .glassBackgroundEffect(in: .capsule)
        }
    }

    func randomRotation() -> Double {
        Double.random(in: 360...720)
    }
}
```

### 17:30 - RealityView
```swift
RealityView { content in
    if let earth = try? await
        ModelEntity(named: "Earth")
    {
       earth.addImageBasedLighting()
       content.add(earth)
    }
}
```

### 18:57 - RealityView Gesture
```swift
struct Earth: View {
		@State private var pinLocation: GlobeLocation?

    var body: some View {
        RealityView { content in
            if let earth = try? await
                ModelEntity(named: "Earth")
            {
               earth.addImageBasedLighting()
               content.add(earth)
            }
        }
				.gesture(
            SpatialTapGesture()
                .targetedToAnyEntity()
                .onEnded { value in
                    withAnimation(.bouncy) {
                        rotation.degrees += randomRotation()
                        animatingRotation = true
                    } completion: {
                        animatingRotation = false
                    }
                    pinLocation = lookUpLocation(at: value)
                }
        )
    }
}
```

### 19:34 - RealityView Attachments
```swift
struct Earth: View {
		@State private var pinLocation: GlobeLocation?

    var body: some View {
        RealityView { content in
            if let earth = try? await
                ModelEntity(named: "Earth")
            {
               earth.addImageBasedLighting()
               content.add(earth)
            }
        } update: { content, attachments in
            if let pin = attachments.entity(for: "pin") {
                content.add(pin)
                placePin(pin)
            }
        } attachments: {
            if let pinLocation {
                GlobePin(pinLocation: pinLocation)
                    .tag("pin")
            }
        }
				.gesture(
            SpatialTapGesture()
                .targetedToAnyEntity()
                .onEnded { value in
                    withAnimation(.bouncy) {
                        rotation.degrees += randomRotation()
                        animatingRotation = true
                    } completion: {
                        animatingRotation = false
                    }
                    pinLocation = lookUpLocation(at: value)
                }
        )
    }
}
```

### 21:11 - ImmersiveSpace
```swift
@main
struct WorldApp: App {
    var body: some Scene {
				// (other WindowGroup scenes)

        ImmersiveSpace(id: "solar-system") {
            SolarSystem()
        }
    }
}
```

### 21:25 - Open ImmersiveSpace Action
```swift
@Environment(\.openImmersiveSpace)
private var openImmersiveSpace

Button("View Outer Space") {
    openImmersiveSpace(id: "solar-system")
}
```

### 22:50 - ImmersionStyle
```swift
@main
struct WorldApp: App {
    @State private var selectedStyle: ImmersionStyle = .full
    var body: some Scene {
				// (other WindowGroup scenes)

        ImmersiveSpace(id: "solar-system") {
            SolarSystem()
        }
        .immersionStyle(selection: $selectedStyle, in: .full)
    }
}
```

### 23:17 - Starfield
```swift
struct Starfield: View {
    var body: some View {
        RealityView { content in
            let starfield = await loadStarfield()
            content.add(starfield)
        }
    }
}
```

### 23:28 - SolarSystem
```swift
struct SolarSystem: View {
    var body: some View {
        Earth()
        Sun()
      	Starfield()
    }
}```
# Resources
* https://developer.apple.com/documentation/visionOS/World
