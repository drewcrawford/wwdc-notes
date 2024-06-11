Learn how to create captivating immersive experiences with ARKit's latest features. Explore ways to use room tracking and object tracking to further engage with your surroundings. We'll also share how your app can react to changes in your environment's lighting on this platform. Discover improvements in hand tracking and plane detection which can make your spatial experiences more intuitive.

### 3:35 - RoomTrackingProvider
```swift
// RoomTrackingProvider
@available(visionOS, introduced: 2.0)
public final class RoomTrackingProvider: DataProvider, Sendable {
    /// The room which a person is currently in, if any.
    public var currentRoomAnchor: RoomAnchor? { get }
    
    /// An async sequence of all anchor updates.
    public var anchorUpdates: AnchorUpdateSequence<RoomAnchor> { get }
    
    ...
}
```

### 4:20 - RoomAnchor
```swift
// RoomAnchor
@available(visionOS, introduced: 2.0)
public struct RoomAnchor: Anchor, Sendable, Equatable {
    /// True if this is the room which a person is currently in.
    public var isCurrentRoom: Bool { get }
    
    /// Get the geometry of the mesh in the anchor's coordinate system.
    public var geometry: MeshAnchor.Geometry { get }
    
    /// Get disjoint mesh geometries of a given classification.
    public func geometries(of classification: MeshAnchor.MeshClassification) -> [MeshAnchor.Geometry]
    
    /// True if this room contains the given point.
    public func contains(_ point: SIMD3<Float>) -> Bool
    
    /// Get the IDs of the plane anchors associated with this room.
    public var planeAnchorIDs: [UUID] { get }
    
    /// Get the IDs of the mesh anchors associated with this room.
    public var meshAnchorIDs: [UUID] { get }
}
```

### 8:06 - Load Object Tracking reference object
```swift
// Object tracking
Task {
    do {
        let url = URL(fileURLWithPath: "/path/to/globe.referenceobject")
        let referenceObject = try await ReferenceObject(from: url)
        let objectTracking = ObjectTrackingProvider(referenceObjects: [referenceObject])
    } catch {
        // Handle reference object loading error.
    }
    ...
}
```

### 8:27 - Run ARKitSession with ObjectTracking provider
```swift
let session = ARKitSession()
Task {
    do {
        try await session.run([objectTracking])
    } catch {
        // Handle session run error.
    }
    
    for await event in session.events {
        switch event {
        case .dataProviderStateChanged(_, newState: let newState, _):
            if newState == .running {
                // Ready to start processing anchor updates.
            }
            ...
        }
    }
}
```

### 8:43 - ObjectAnchor
```swift
// ObjectAnchor
@available(visionOS, introduced: 2.0)
public struct ObjectAnchor: TrackableAnchor, Sendable, Equatable {
    /// An axis-aligned bounding box.
    public struct AxisAlignedBoundingBox: Sendable, Equatable {
        ...
    }
    
    /// The bounding box of this anchor.
    public var boundingBox: AxisAlignedBoundingBox { get }
    
    /// The reference object which this anchor corresponds to.
    public var referenceObject: ReferenceObject { get }
}
```

### 11:03 - World Tracking - reacting to changes in lighting conditions
```swift
struct WellPreparedView: View {
    @Environment(\.worldTrackingLimitations) var worldTrackingLimitations
    
    var body: some View {
        ...
        
        .onChange(of: worldTrackingLimitations) { 
            if worldTrackingLimitations.contains(.translation) {
                // Rearrange content when anchored positions are unavailable.
            }
        }
    }
}
```

### 12:51 - Hands prediction
```swift
// Hands prediction
func submitFrame(_ frame: LayerRenderer.Frame) {
    ...
    
    guard let drawable = frame.queryDrawable() else { return }
    
    // Get the trackable anchor time to target.
    let trackableAnchorTime = drawable.frameTiming.trackableAnchorTime
    
    // Convert the timestamp into units of seconds.
    let anchorPredictionTime = LayerRenderer.Clock.Instant.epoch.duration(to: trackableAnchorTime).timeInterval
    
    // Predict hand anchors for the time that provides best content registration.
    let (leftHand, rightHand) = handTracking.handAnchors(at: anchorPredictionTime)
    ...
}
```

# Resources
https://developer.apple.com/documentation/arkit/arkit_in_visionos/building_local_experiences_with_room_tracking
