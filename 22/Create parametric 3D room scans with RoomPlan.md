Apple has enabled powerful new ways for peopel to bring the world into their apps.

Last year we introduced Object Capture.  photogrammetry API.

Previous to OC, we released scene reconstruction API, which gives you a coarsce understanding of geometric structure of your scene, new augmented-reality usecases.

This year we're announcing #roomplan framework.

# RoomPlan
Scan your room using LiDAR-enabled  ipHone or ipad.  Parametric 3d model of the room and its room-deifning objects which you can use in your app.

Detect wall,s iwndows, opening,s, and doors, as well as room-defining objects like fireplaces, couches, tables, and cabinets.  With our API, you can integrate a scanning experience into your app.

RoomCaptureView presents the final, post-processed result.

# Applications
* Interior design
* Architecture
* Real estate
* E-commerce

Integrate RP into your app.  Let's take a look.

# Scanning experience API
## RoomCaptureView
* world space feedback
* realtime model generation
* coaching and user guidance
During an active session, animated olines outline detected walls, windows, openings, doors, and room-deifning objects in realtime.  Interactive 3D model generated at the bottom gives you an overview of your progress at a glance.  Text coaching guides you to best scan results.

```swift
// RoomCaptureView API - Scan & Process

import UIKit
import RoomPlan

class RoomCaptureViewController: UIViewController {

    var roomCaptureView: RoomCaptureView
    var captureSessionConfig: RoomCaptureSession.Configuration

   private func startSession() {
        roomCaptureView?.captureSession.run(configuration: captureSessionConfig)
     }
    
    private func stopSession() {
        roomCaptureView?.captureSession.stop()
	}
}
```

export as usdz, etc.

```swift
// RoomCaptureView API - Export

import UIKit
import RoomPlan

class RoomCaptureViewController: UIViewController {
  …
  func captureView(shouldPresent roomDataForProcessing: CapturedRoomData, error: Error?) -> Bool {
    // Optionally opt out of post processed scan results.
    return false
  }

  func captureView(didPresent processedResult: CapturedRoom, error: Error?) {
    // Handle final, post processed results and optional error.
    // Export processedResults
    …
    try processedResult.export(to: destinationURL)
    …
  }
}
```



# Data API
Access to underlying datas from scanning.

## Basic workflow
1.  Scan
2. process
3. export

## Scanning
We will use RoomCaptureSession API to set up and start, and display the progress.


```swift
import UIKit
import RealityKit
import RoomPlan
import ARKit

class ViewController: UIViewController {
    @IBOutlet weak var arView: ARView!
    var previewVisualizer: Visualizer!
    lazy var captureSession: RoomCaptureSession = {
        let captureSession = RoomCaptureSession()
        arView.session = captureSession.arSession
        return captureSession
    }()
    override func viewDidLoad() {
        super.viewDidLoad()
        captureSession.delegate = self
        // set up previewVisualizer
    }
}
```

```swift
// Getting live results and user instructions

extension ViewController: RoomCaptureSessionDelegate {

    func captureSession(_ session: RoomCaptureSession,
                        didUpdate room: CapturedRoom) {
        previewVisualizer.update(model: room)
    }
    
    func captureSession(_ session: RoomCaptureSession,
                   didProvide instruction: Instruction) {
        previewVisualizer.provide(instruction)
    }
}
```

Instructions
* move closer to wall
* away from wall
* slow down
* turn on light
* low texture
## Process
Use RoomBuilder to process scanned data and generate 3d models.

```swift
// RoomBuilder
import UIKit
import RealityKit
import RoomPlan
import ARKit

class ViewController: UIViewController {
    
    @IBOutlet weak var arView: ARView!
    var previewVisualizer: Visualizer!

    // set up RoomBuilder
    var roomBuilder = RoomBuilder(options: [.beautifyObjects])
}
```

```swift
// RoomBuilder with the latest CapturedRoomData to generate final 3D CapturedRoom

extension ViewController: RoomCaptureSessionDelegate
{
    func captureSession(_ session: RoomCaptureSession, 
                        didEndWith data: CapturedRoomData, error: Error?)
    {
        if let error = error {
            print("Error: \(error)")
        }
        Task {
            let finalRoom = try! await roomBuilder.capturedRoom(from: data)
            previewVisualizer.update(model: finalRoom)
        }
    }
}
```

Model will be available for final presentation.
## Export
At top level, capturedRoom consists of surfaces and objects.

Surface contains curve,edge,category
curve (radius, angles)
edge => 9left,right,top,bottom)
category => wall,opening,window,door

object => category
category => table,bed,sofa

surface ando bjects share common attributes => dimensions,confidence,transform,identifier

```swift
// CapturedRoom and export

public struct CapturedRoom: Codable, Sendable
{
    public let walls: [Surface]
    public let doors: [Surface]
    public let windows: [Surface]
    public let openings: [Surface]
    public let objects: [Object]

    public func export(to url: URL) throws

    // Surface definitions ...
    
    // Object definitions ...
}
```

The first four elements are represented as a `Surface` structure.  Various properties, etc.

Last property is an array of `Object`, which are another struct.  Many objects we support.  Common furniture types.

export function lets you export to USD Or USDZ.  
# Best practices
* Recommended conditions
* Room selection
* Scanning considerations
* Thermal considerations
## Recommended conditions
* Single residential room
* 30ftx30ft.
* Lighting: minimum 50 lux
* Lidar-enabled iPhone and iPad Pro models
## Room selection
Challenging conditions:
* full-height mirrors or glass
* High ceilings
* Dark surfaces
## Scanning considerations
* Preparing your room
	* Opening the curtains (improve lighting, window occludance)
	* Closing doors (don't scan unnecessary areas)
* Instruction delegate
	* Texture
	* distance
	* speed
	* lighting

## Thermal considerations
Challenging conditions
* repeated scans
* long scans over 5 minutes
# Wrap up
* Intuitive scannign experience
* Powerful data API for scene understanding
* Fully parameteric USD representations
[[Qualities of a great AR experience]]
[[Dive into RealityKit 2]]


* https://developer.apple.com/forums/tags/wwdc2022-10127
* https://developer.apple.com/forums/create/question?&tag1=361030&tag2=409030
* https://developer.apple.com/documentation/roomplan/create_a_3d_model_of_an_interior_room_by_guiding_the_user_through_an_ar_experience
* https://developer.apple.com/documentation/RoomPlan