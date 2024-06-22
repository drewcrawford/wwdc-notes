Build a board game for visionOS from scratch using TabletopKit. We'll show you how to set up your game, add powerful rendering using RealityKit, and enable multiplayer using spatial Personas in FaceTime with only a few extra lines of code.

### 3:52 - Make a rectangular table
```swift
// Make a rectangular table.
let entity = try! Entity.load(named: "table", in: table_Top_KitBundle)
let table: Tabletop = .rectangular(entity: entity)
```

### 4:25 - Place seats
```swift
// Place 3 seats around the table, facing the center.
static let seatPoses: [TableVisualState.Pose2D] = [
    .init(position: .init(x: 0, y: Double(GameMetrics.tableDimensions.z)), rotation: .degrees(0)),
    .init(position: .init(x: -Double(GameMetrics.tableDimensions.x), y: 0), rotation: .degrees(-90)),
    .init(position: .init(x: Double(GameMetrics.tableDimensions.x), y: 0), rotation: .degrees(90))
]
```

### 5:40 - Define player pawns
```swift
// Define an object that describes a pawn for each player.
struct PlayerPawn: EntityEquipment {
    let id: ID
    let entity: Entity
    var initialState: BaseEquipmentState
    
    init(id: ID, seat: PlayerSeat, pose: TableVisualState.Pose2D, entity: Entity) {
        self.id = id
        self.entity = entity
        initialState = .init(seatControl: .restricted([seat.id]), pose: pose, entity: entity)
    }
}
```

### 6:55 - Define an object that describes a tile
```swift
// Define an object that describes a tile on the conveyor belt
struct ConveyorTile: Equipment {
    enum Category: String {
        case red
        case green
        case grey
    }
    
    let id: ID
    let category: ConveyorTile.Category
    let initialState: BaseEquipmentState
    
    init(id: ID, boardID: EquipmentIdentifier, position: TableVisualState.Point2D, category: ConveyorTile.Category) {
        self.id = id
        self.category = category
        initialState = .init(parentID: boardID, pose: .init(position: position, rotation: .init()), boundingBox: .init(center: .zero, size: .init(x: 0.06, y: 0, z: 0.06)))
    }
}
```

### 9:53 - Monitor interactions
```swift
// The view contains all the content in the game.
RealityView { (content: inout RealityViewContent) in
    content.entities.append(loadedGame.renderer.root)
}.tabletopGame(loadedGame.tabletop, parent: loadedGame.renderer.root) { _ in
    GameInteraction(game: loadedGame)
}

// Define an object that manages player interactions.
struct GameInteraction: TabletopInteraction {
    func update(context: TabletopKit.TabletopInteractionContext, value: TabletopKit.TabletopInteractionValue) {
        switch value.phase {
            //...
        }
    }
}
```

### 10:47 - Respond to interaction updates
```swift
// Insert code snippet.
```

### 12:52 - Add a sound effect to the die roll
```swift
// Respond to interaction updates.
func update(context: TabletopKit.TabletopInteractionContext, value: TabletopKit.TabletopInteractionValue) {
    switch value.gesturePhase {
        //...
        case .ended: {
            if let die = game.tabletop.equipment(of: Die.self, matching: value.startingEquipmentID) {
                if let audioLibraryComponent = die.entity.components[AudioLibraryComponent.self] {
                    if let soundResource = audioLibraryComponent.resources["dieSoundShort.mp3"] {
                        die.entity.playAudio(soundResource)
                    }
                }
            }
        }
        //...
    }
}
```

### 14:44 - Set up multiplayer with SharePlay
```swift
// Set up multiplayer using SharePlay.

// Provide a button to begin SharePlay.
import GroupActivities

func shareplayButton() -> some View {
    Button("SharePlay", systemImage: "shareplay") {
        Task { try! await Activity().activate() }
    }
}

// After joining the SharePlay session, start multiplayer.
sessionTask = Task.detached { [weak self] @MainActor in
    for await session in Activity.sessions() {
        self?.tabletopGame.coordinateWithSession(session)
    }
}
```
# Resources
* https://developer.apple.com/documentation/TabletopKit/TabletopKitSample
* https://developer.apple.com/documentation/TabletopKit/TabletopKitSample
