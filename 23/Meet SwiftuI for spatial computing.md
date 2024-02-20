Take a tour of the solar system with us and explore SwiftUI for visionOS! Discover how you can build an entirely new universe of apps with windows, volumes, and spaces. We'll show you how to get started with SwiftUI on this platform as we build an astronomy app, add 3D content, and create a fully immersive experience to transport people to the stars.

Bringing swiftui into a bold new future.  Volumes, etc.

home view, TV, etc.  SAfari.  All-new experiences like 3D in freeform.  Immersive rehearsals in keynote.  SwiftuI powers all of these things and more!

describe your app's UX, leaving the system to apply defaults for you.  Even more useful with an entirely new platform.  Existing knowledge tranfers over seamlessly.

buttons on the system have a lot in common with buttons you know/love on other platforms.

macOS - bordered style by default.

### 2:02 - Button
```swift
Button("Of course") {
  // perform action
}
```

key differences.

all buttons gain rich hover effects that react to eyes, hands, etc.
scale down and provide audio feedback when pressed.
navigation bars - display tooltip when people look.

consider tabview.  expands to display more detail

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

same thoughtful design to all core building blocks.  Navigation, presentation, controls, interactions.  Intelilgent defaults so you can fit in from the start.

a whole new suite of brand-new APIs built for a new environment.

scenes.

made up of views.

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

# Windows
traditional apps, safari, etc.  

WindowGroup - supports multiple windows, etc.

* glass background
* tab view
* navigation stacks, split views
* lists
* interactivity using builtin controls, buttons, toggles, pickers, etc.
* tab view - make toggleable entrypoints accessible at all times
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

unique appearance, hangs off the edge.  Ornaments.  
* place accessories outside the windows' bounds
* used in tabview, presentations, and more
* make your own using the `.ornament` modifier

glass adapts to the environment, etc.

no dark/light apeparance.  Materials do the hard work for you.  How to use to make them fit in and improve legibility?
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

card is built as a button, showing more details when I press it.  Custom buton style, title, detial body, and footnote.  

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

interaction.  New ways to interact with apps.

* indirect pinch gesture
* direct input
* pointer
* AX

gestures you're already familiar with work great.  Automatic interaction, etc.

brand new gestuers to enable new kinds of rich 3D interactions.  RotateGesture3D.  Using two hands or connected trackpad.

All the same AX APIs from other platforms.  VO, rotoers, dynamic type, etc.  many features are reimagined.  Dwell control - using only their eyes.

To learn more - [[Create accessible spatial experiences]]

hover effects
* critical to making your app responsive
* run outside of your app's process
* added automatically to most controls

add to your custom controls

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

* add ornaments to your app
* dive deeper on hover effects, materials, and more
[[Elevate your windowed app for spatial computing]]




# Volumes
new 3D window style, objects/experiences in the bounded space.  Alongside other apps, 3d experiences, etc.

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

"just another swiftui view".  Same concepts you're already familiar with.  natural extensions to the layout system, VFX, gestures, etc.

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

Easy way to spin the globe in place.  My favorite way to plan my next vacation.  Rotation effect.

RealityView

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


display my earth model by creating a model entity.  Once it finishes loading, add its ocntent to display.  async/await to add model3d to display placeholder, etc.

* use realitykit in your swiftui views
* add rich behaviors with custom materials, shaders, physics, and more

[[Build spatial experiences with RealityKit]]
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


add a swiftUI views to a realityview with attachments.  Place as RK entity anywhere in realityview.


[[Take SwiftUI to the next dimension]]

# Full spaces
full spaces - brand new way to build rich, immersive apps.  Hide windows from other apps, place your content anywhere, etc.

Use multiple scenes of the same type, to organize your app into separate pieces.  Let's dig into each one.

hiding surroundings to create stunning new experiences.  Solar systems, etc.

provide a root view for the space's context.  id so i can programmatically open from my main window.

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

then open by id
### 21:25 - Open ImmersiveSpace Action
```swift
@Environment(\.openImmersiveSpace)
private var openImmersiveSpace

Button("View Outer Space") {
    openImmersiveSpace(id: "solar-system")
}
```

A full space comes in one of several immersion styles.

mixed - coexist with real world
full - fully immersive and hide surroundings
progressive - middle ground.  use the digital crown to dial in how much immersion feels right.


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
Use ARKit.  Powerful framework deep in the system.
* gain real understanding of real world
* place content near physical surfaces
* build powerful hand gestures

[[Meet ARKit for spatial computing]]

by integrating ARKit, I was able to implement anew hand gesture to summon the earth.  

[[Go beyond the window with SwiftUI]]
Take full control with Metal
[[Discover Metal for immersive apps]]

# Wrap up
dive deeper
[[Principles of spatial design]]
[[Meet UIKit for spatial computing]]

# Resources
* https://developer.apple.com/documentation/visionOS/World
