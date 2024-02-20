Discover how you can use the GroupActivities framework to build unique sharing and collaboration experiences for visionOS. We'll introduce you to SharePlay on this platform, learn how to create experiences that make people feel present as if they were in the same space, and explore how immersive apps can respect shared context between participants.

In Facetime, they all take up space in your room.  Together with others.

We introduced shared context.  Everyoen will see everyone else in the same relative position.

SharePlay as well.  Same relative position for everyone.  Templates are used to determine how participants will be arranged relative to each other, spatial consistency. 

Apps should worry about visual consistency.  Same content inside the app.  Imagine being in front of a whiteboard with others.  Everyone has same view of whiteboard.  We want to replicate this in shareplay.  Basically the app should keep content in sync.

[[Design spatial SharePlay experiences]]
# Windowed apps

Receive system state
provide configurations during shareplay

ex, don't sync scrolling maybe?

To observe if you're spatial, you first need to get system coordiantor for youp session.


### 4:08 - Observe the local participant state
```swift
for await session in ExploreActivity.sessions() {
    guard let systemCoordinator = await session.systemCoordinator else { continue }

    let isLocalParticipantSpatial = systemCoordinator.localParticipantState.isSpatial

    Task.detached {
        for await localParticipantState in systemCoordinator.localParticipantStates {
            if localParticipantState.isSpatial {
                // Start syncing scroll position
            } else {
                // Stop syncing scroll position
            }
        }
    }
}
```

templates, we support 3 different templates
* side by side, in an arc towards shared app.  Default.
* Conversational.  Half-circle with app in front.  Experiences where the focus isn't on the content of the app, such as music.
* Surround.  In a circle, app in center.  Only available for volumetric scenes.
Distance is based on the size of the app.  If app is bigger, placed further from the people.

3 preferences available for SP apps
* `.none` -> system behavior.  SxS for vertical apps, and surround for volumetric apps
* `.sideBySide` applies to provide sxs template
* `.conversational` -> corresponding template.

### 6:10 - Configure the spatial template preferences
```swift
for await session in ExploreActivity.sessions() {
    guard let systemCoordinator = await session.systemCoordinator else { continue }

    var configuration = SystemCoordinator.Configuration()
    configuration.spatialTemplatePreference = .sideBySide
    systemCoordinator.configuration = configuration

    session.join()
}
```

What if your app has multiple window scenes and more than one is currently foregrounded?  Scene on the left is a browse view.  Small scene in the middle provides an easy way to navigate through content.  last scene is a detail view.

All these can be open at the same time.  When in SP, we want the detail scene to be sahred, but if all of them are open, the worng scene could be used in the template.  So solve, scene association tells system which scene is hosting the SP activity.

1.  Share menu above a window to show to the person which scene is shared.  Helpful indicator.
2. Determines which window scene will be used with the template.  Since only one scene is possible, we will automatically associate that scene with the group session.
	3. Otherwise a random scene will be selected!!

When group activity gets activated, we go through each scene to find which one can handle this, by evaluating the activity condition against identifier of group activity.

1.  Can -> scene can handle it
2. Prefers -> directs to more ideally suited.

In our example.  List view can , but doesn't prefer.
Middle nav is neither
Last scene can and prefers.  So we associate that one with the group session.

If no scene can handle the identifier we launch a new scene.  Great if you require your own scen to work in.  If multiple can and none prefer, we pick a can at random.

use `.handlesExternalEvents` with preferring and allowing strings.

```swift
@main
struct ExploreTogetherApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .handlesExternalEvents(
                    preferring: ["com.example.explore-together.activity"],
                    allowing: ["com.example.explore-together.activity"]
                )
        }
    }
}
```


In UIKit, we specify predicates.
```swift
class SceneDelegate: NSObject, UISceneDelegate {

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // ...

        scene.activationConditions.canActivateForTargetContentIdentifierPredicate =
                NSPredicate(format: "self == %@", "com.example.explore-together.activity")

        scene.activationConditions.prefersToActivateForTargetContentIdentifierPredicate =
                NSPredicate(format: "self == %@", "com.example.explore-together.activity")
    }
}
```

[[Targeting content with Multiple Windows]]

Let's consider a document-based app where each scene represents a different document.  Group activity identifier to evaluate each of the scene's activation conditions.  Nothing inherently different, and the wrong scene gets associated.  In this case we want to change the identifier.

We could use a unique identifier of each document.  Then we use the identifier to find the right scenes.

```swift
struct ExploreActivity: GroupActivity {
    var metadata: GroupActivityMetadata {
        var metadata = GroupActivityMetadata()
        // ...
        metadata.sceneAssociationBehavior = .content("document-1")
        return metadata
    }
}
```

Keep in mind that same id will be used for all participants of session.

3 types
* default -> identifier that is used will be the group activity id.  
* Content -> specify a custom identifier.  Meant for apps where each scenes will show different content and gruop activity is tied to that content
* None -> disable scene association.  No scene will be associated os the person won't see the shared banner above any scenes of your app.  Spatial consistency is broken.  Only use in specific cases like immersive apps that don't need an additional scene, or each scene is different, etc.

How to find out which scene we got associated with?  New property on the gruop session called `sceneSessionIdentifier`.  Includes the identifier of the... that was associated.

Share entire window.  Everyone else sees video feed of the window.  SP app can take this further, by exposing a group activity.  Surfaced in the menu.  Improve the discoverability of your SP activity and can eliminate the need to have a dedicated SP button.

```swift
// Create the activity
let activity = ExploreActivity()

// Register the activity on the item provider
let itemProvider = NSItemProvider()
itemProvider.registerGroupActivity(activity)

// Create the activity items configuration
let configuration = UIActivityItemsConfiguration(itemProviders: [itemProvider])

// Provide the metadata for the group activity
configuration.metadataProvider = { key in
    guard key == .linkPresentationMetadata else { return nil }
    let metadata = LPLinkMetadata()
    metadata.title = "Explore Together"
    metadata.imageProvider = NSItemProvider(object: UIImage(named: "explore-activity")!)
    return metadata
}
self.activityItemsConfiguration = configuration
```

system goes through UI responder chain looking for group activity.  
# Immersive apps

System bgs all other apps.  Can be fully focused in this app.

[[Go beyond the window with SwiftUI]]

During FT, you can launch an immersive space at any time.  When in your private immersive space, you're in your own private world, and you break shared context with everyone else.  System hides other people.  They see your contact photo.


### 16:03 - Configure group ImmersiveSpace
```swift
for await session in ExploreActivity.sessions() {
    guard let systemCoordinator = await session.systemCoordinator else { continue }

    var configuration = SystemCoordinator.Configuration()
    configuration.supportsGroupImmersiveSpace = true
    systemCoordinator.configuration = configuration
}
```

 System moves space origin to group space.  Spatial consistency, etc.  people in the group space will see each other as personas.  Shared coordinate system and people in there.  We can place objects at the same location in it to get spatial consistency.
We can place a globe in the center above origin and make sure we apply the same offset for everyone.

How to maintain visualc onsistency?  Ex if we want to rotate the globe, we need to sync this position in some way.

To learn more, [[Build custom experiences with Group Activities]]

We can also add UI elements relative to each person in group space.  ex, give every participant a private control menu that's right in front of them.  Manipulate the shared globe easily.  

Must know where local user is.

### 17:51 - System Experience Displacement
```swift
// Use immersiveSpaceDisplacement to offset contents in group immersive space

var body: some Scene {
    ImmersiveSpace(id: "earth") {
        GeometryReader3D { proxy in
            let displacement = proxy.immersiveSpaceDisplacement(in: .global).inverse

            Control()
                .offset(displacement.position)
                .rotation3DEffect(displacement.rotation)
        }
    }
}
```

Note that the displacement does not update if the user moves.  It only provides a displacement of space when it was initially placed.  Easy way to place content relative to the person without needing to fully track where they are.

By default when app only has a group immersive space, we use surround.  Same layout is applied when your app also has a shared volumetric scene.  But if you have a shared vertical window, we use sxs template.  You can change template preference.  Choose `.conversational` template or sxs template.  

Distances in template are dynamically adjusted.  How to ensrue we have good  distance with only group immersive space?
### 20:46 - Spatial Template Preferences
```swift
// Configure the spatial template preferences with content extent

for await session in ExploreSolarActivity.sessions() {
    guard let systemCoordinator = await session.systemCoordinator else { continue }

    var configuration = SystemCoordinator.Configuration()
    configuration.supportsGroupImmersiveSpace = true
    configuration.spatialTemplatePreference = .sideBySide.contentExtent(200)
    systemCoordinator.configuration = configuration
}
```

apps set this value as distance from center of content, to farthest edge in points.  Active template will use this value.  

Problems with mixing immersive styles, etc.  Ideally your friends can just follow you into immersive space automatically.  To minimize split contacts, we have a tool.  `groupImmersionStyle`.

Provides an async sequence of optional immersive styles which tells you what people join?

ex when mia opens a space, connor gets that space via sequence.  When connor leaves, mia gets nil for groupImmersionStyle.

Now with the help of the groupImmersionStyle, easier to be together.


### 22:32 - Receive group immersion style to configure group immersive space
```swift
// Receive group immersion style to configure group immersive space

for await session in ExploreSolarActivity.sessions() {
    guard let systemCoordinator = await session.systemCoordinator else { continue }

    Task.detached {
        for await immersionStyle in systemCoordinator.groupImmersionStyle {
            if let immersionStyle {
                // Open an immersive space with the same immersion style
            } else {
                // Dismiss the immersive space
            }
        }
    }
}
```

May want to step out temporarily.  Digital crown.  System will not disturb other people by changing their group immersion style.  Tells you where the rest of the group is with a button to rejoin.  When you tap on the joint button, a group immersion style will be sent to your app.  

# Wrap up
* spatial and visual consistency
* Template preferences
* SharePlay from Share menu


# Resources
* https://developer.apple.com/documentation/GroupActivities/SystemCoordinator