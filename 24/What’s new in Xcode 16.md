Discover the latest productivity and performance improvements in Xcode 16. Learn about enhancements to code completion, diagnostics, and Xcode Previews. Find out more about updates in builds and explore improvements in debugging and Instruments.

# Edit

on-device coding model.

**on macOS Sequoia**.

Incrementally enable warnings for each new language feature.  "Upcoming features".

[[Migrate your app to Swift 6]]

`@Previewable`.  Wrapper views no longer necessary.

`PreviewModifier`.  Preview system can cache the data.

### 3:37 - Inline State within Preview
```swift
#Preview { @Previewable @State var currentFace = RobotFace.heart }
```

### 3:45 - View using Inline State
```swift
RobotFaceSelectorView(currentFace: $currentFace)
```

### 3:53 - Complete Preview using Previewable
```swift
#Preview { @Previewable @State var currentFace = RobotFace.heart RobotFaceSelectorView(currentFace: $currentFace) }
```

### 4:40 - Type Conforming to PreviewModifier
```swift
struct SampleRobotNamer: PreviewModifier {
    typealias Context = RobotNamer
    static func makeSharedContext() async throws -> Context {
        let url = URL(fileURLWithPath: "/tmp/local_names.txt")
        return try await RobotNamer(url: url)
    }
    func body(content: Content, context: Context) -> some View {
        content.environment(context)
    }
}
```

### 5:29 - Extension on PreviewTrait
```swift
extension PreviewTrait where T == Preview.ViewTraits {
    @MainActor static var sampleNamer: Self = .modifier(SampleRobotNamer())
}
```

### 5:38 - Preview using created PreviewModifier
```swift
#Preview(traits: .sampleNamer) { RobotNameSelectorView() }
```


# Build

Explicit Modules.
Improved parallelism, better diagnostics, faster debugging.

C/ObjC: On by default
Swift: Opt-in


Xcode splits up compilation unit into 3 phases
* scan ("scan dependency")
* build modules ("compile swift module / compile clang module")
* build source

[[Demystify explicitly built modules]]


# Debug

Faster debugigng with explicit modules
smaller, faster, debug systems with DWARF5 on macOS Sequoia and iOS 18 (DT required)

## thread performance checker
* main thread hangs
* priority inversion
* disk write diagnostics (new)
* launch diagnostics (new)

organizer -> launches.  

### 10:26 - AVPlayer Creation
```swift
struct BOTanistAVPlayer {
    func player(url: URL) throws -> AVPlayer {
        let player = AVPlayer(url: url)
        return player
    }
}
```

### 11:28 - AVPlayer Call Site
```swift
self.player = try? await robotVideoAVPlayer()
```

### 11:57 - AVPlayer Initialization
```swift
private nonisolated func robotVideoAVPlayer() async throws -> AVPlayer? {
    guard let url = Bundle.main.url(forResource: RobotVideo.resource, withExtension: RobotVideo.ext) else {
        throw BOTanistAppError.videoNotFound(forResource: RobotVideo.resource, withExtension: RobotVideo.ext)
    }
    let avPlayer = BOTanistAVPlayer()
    let player = try avPlayer.player(url: url)
    return player
}
```

## realitykit
[[Break into the RealityKit debugger]]
[[Run, break, and inspect Explore effective debugging in LLDB]]

# Test
### 13:42 - Initial Test Scaffolding
```swift
import Testing
@testable import BOTanist

// When using the default init Plant(type:) make sure the planting style is graft
@Test
func plantingRoses() {
    // First create the two Plant structs
    // Verify with #expect
}
```


### 14:36 - Complete Test
```swift
import Testing
@testable import BOTanist

// When using the default init Plant(type:) make sure the planting style is graft
@Test
func plantingRoses() {
    // First create the two Plant structs
    let plant = Plant(type: .rose)
    let expected = Plant(type: .rose, style: .graft)
    
    // Verify with #expect
    #expect(plant == expected)
}
```



### 17:35 - Custom Tag
```swift
extension Tag {
    @Tag static var planting: Self
}
```

### 17:42 - Tag Usage in @Test
```swift
.tags(.planting)
```

[[Meet Swift Testing]]
[[Go further with Swift Testing]]
# Profile

Flame graph.

### 20:37 - Slow Asset Loading
```swift
for asset in allAssets {
    asset.load()
}
```

### 20:54 - Fast Asset Loading
```swift
await withDiscardingTaskGroup { group in
    for asset in allAssets {
        group.addTask { asset.load() }
    }
}
```
# Resources
* https://developer.apple.com/documentation/Xcode/previewing-your-apps-interface-in-xcode
* https://developer.apple.com/xcode/
* https://developer.apple.com/documentation/Updates/Xcode
