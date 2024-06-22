Extend your media viewing experience using Reality Composer Pro components like Docking Region, Reverb, and Virtual Environment Probe. Find out how to further enhance immersion using Reflections, Tint Surroundings Effect, SharePlay, and the Immersive Environment Picker.

### Add environments to the Immersive Environment Picker - 15:14
```swift
WindowGroup {
    ContentView()
        .immersiveEnvironmentPicker {
            ForEach(viewModel.environmentItems) { item in
                Button(item.title, image: item.thumbnail) {
                    Task { await openImmersiveSpace(id: item.id) }
                }
            }
        }
}
```

### Synchronization of an environment state using SharePlay - 15:47
```swift
import AVKit
import GroupActivities

for await session in MyActivity.sessions() {
    // join the session, activate the activity, etc.
    playerViewController.player?.playbackCoordinator.coordinateWithSession(session)
    playerViewController.groupExperienceCoordinator.coordinateWithSession(session)
}
```

# Resources
* https://developer.apple.com/documentation/visionOS/building-an-immersive-media-viewing-experience
* https://developer.apple.com/documentation/visionOS/destination-video
* https://developer.apple.com/documentation/visionOS/enabling-video-reflections-in-an-immersive-environment
* https://developer.apple.com/sample-code/ar/WWDC_2024_Diffuse_Reflection_UV_Computation_Tool.zip

