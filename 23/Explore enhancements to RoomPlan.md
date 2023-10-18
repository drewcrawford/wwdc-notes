Join us for an exciting update to RoomPlan as we explore MultiRoom support and enhancements to room representations. Learn how you can scan areas with more detail, capture multiple rooms, and merge individual scans into one larger structure. We'll also share workflows and best practices when working with RoomPlan results that you want to combine into your existing 3D model library.

Detect walls, windows, doors, openings, etc.

Integrate a scanning experience directly into your app. Once finished scanning, present 3d model, and export to usdz. 

We will talk about what’s new.

# Custom ARSession support

Relying on information from ARKit to detect walls, windows, doors, openings, etc.  

Running with a default arsession.  New in iOS 17, roomplan can use a custom ARSession with world tracking configuration.  New ways to combine roomplan with ARKit.  

* increase immersion
* Add photos and videos to roomplan scans
* Seamlessly integrate roomplan with your ARKit app
* Just a few usecases.

###  Build a new trait collection instance from scratch - 3:00
```swift
// RoomCaptureSession

public class RoomCaptureSession {
    // Init: ARSession is an optional input for RoomCaptureSession
    public init(arSession: ARSession? = nil) {
       ...
  }

    // Stop: pauseARSession is used for whether to continue ARSession experience
    public func stop(pauseARSession: Bool = true) {
       ...
  }
}
```

The stop function improves a new option to determine whether you want to pause the underlying AR session.  


# MultiRoom support

In previous roomplan, do a single scan with a 3d model for a single room.

Say you have done several scans with different rooms, such as a tiny nook, kitchen, hallway, etc.  If you want to merge them, you will face some challenges.  

Even if you stitch them manually, you have duplicated walls, apertures, etc.

Capture for multi room.
* continuous ARSession
* ARSession relocalization

New api has new argument in the stop function where we set to false.  Allows arsession to keep running for the next scan, until we finally decided ot pause session again.

Using this approach we can ensure the same session runs across several scans.  Allows us to have one coordinate system for all scans.

###  MultiRoom support with Continuous ARSession - 5:50
```swift
// Continuous ARSession

// start 1st scan
roomCaptureSession.run(configuration: captureSessionConfig)

// stop 1st scan with continuing ARSession
roomCaptureSession.stop(pauseARSession: false)

// start 2nd scan
roomCaptureSession.run(configuration: captureSessionConfig)

// stop 2nd scan (pauseARSession = true by default)
roomCaptureSession.stop()
```

* continuous ARSession
* ARSession relocalization
	* Suitable for individual room scans at different times

###  MultiRoom capture with loading ARWorldMap - 7:30
```swift
// Capture with loading ARWorldMap

// load ARWorldMap
let arWorldMap = try NSKeyedUnarchiver.unarchivedObject(ofClass: ARWorldMap.self, from: data)

// run ARKit relocalization
let arWorldTrackingConfig = ARWorldTrackingConfiguration()
arWorldTrackingConfig.initialWorldMap = arWorldMap
roomCaptureSession.init()
roomCaptureSession.arSession.run(arWorldTrackingConfig, options: [])

// Wait for relocalization to complete
// start 2nd scan
roomCaptureSession.run(configuration: captureSessionConfig)

// stop 2nd scan
roomCaptureSession.stop()
```

How to merge into 1 combined structure?  RoomBuilder.  

Now that all captured rooms are in same space, we use StructureBuilder.

###  StructureBuilder - 9:40
```swift
// StructureBuilder

// create structureBuilder instance
let structureBuilder = StructureBuilder(option: [.beautifyObjects])

// load multiple capturedRoom results to capturedRoomArray
var capturedRoomArray: [CapturedRoom] = []

// run structureBuilder API to get capturedStructure
let capturedStructure = try await structureBuilder.capturedStructure(from: capturedRoomArray)

// export capturedStructure to usdz
try capturedStructure.export(to: destinationURL)
```

Merge multiple scans.  Structure builder instance with configuration options.  

Clear array of captured rooms.  After that, we call structurebuilder to get captured structure.  

Finally, we export to usdz.

Walls, doors, windows, openings, objects, etc.

Function to export to usdz.

###  CapturedStructure - 10:11
```swift
// CapturedStructure

public struct CapturedStructure: Codable, Sendable {
    public var rooms: [CapturedRoom]

    public var walls: [Surface]
    public var doors: [Surface]
    public var windows: [Surface]
    public var openings: [Surface]
    public var objects: [Object]
    public var floors: [Surface]
    public var sections: [Section]

    public func export(to url: URL, metadataURL: URL? = nil, modelProvider: ModelProvider? = nil, exportOptions: USDExportOptions = .mesh) throws
}
```

Our usdz file is ready to preview in both iOS and macOS.  Take this one step further by loading to a digital content creation tool such as a blender.

## considerations
* single floor residential house
* 1-4 bedrooms, living room, kitchen, dining room
* House area: 2k sqft
* lighting: minimum 50lu
# Accessibility

Most of us think about visual rendering.  But for low-vision people, it’s a problem

Audio feedback to have guidelines about scanning and what we see.



# Representation improvements

Makes it easy to scan a wide range of rooms.  It was limited to accurately represent limited situations.  Now w e support a larger variety of rooms.  Slanted walls, recessed kitchen elements, dishwashers, etc.

Many types of sofas, etc.

Surfaces, objects.  Now we have sections.

Walls can be described as polygons for no uniform walls.  

Besides surface categories, we added floor categories.  Also as apolygon.

Objects now have attributes to describe different configurations.

Surfaces and objects have a parent variable.  Ex, a parent of a window is a wall, parent of a dishwasher can be a storage?

Given position, given floor.  No uniform walls can be rendered as polygons.

Floor is now represented as a rectangle during scanning, and beautified as a polygon.

When we introduced roomplan, categories were used to describe an object.  But this has limitations.  Ex, chair.  Several types of chairs can represent this category, stool, dining chair, etc.

We are now adding attributes. Ex with back, without arms, etc.



# Enhanced export functions

Can now be included in exported results.  
* UUId mapping
* Model provider

When exporting as a mesh, we rendered as a tree of nodes.  This year, we added a section group containing section centers.  But we’re missing a substantial amount of information, such as dimensions of walls and objects, attributes, etc.

Mapping file can now be created.  Dictionary of string to UUID, between node name and room elements.  Uniquely identified by the identifier.

Previously, a room was exported in two ways
* usdz
* JSON or list, encoding captured rooms tructure

New in iOS 17, can relate the two exported information.  When rendering your scan, you can query additional info about an object.

Model provider.  Now produces models that match the attributes.

Replace bounding box with more compelling rendering.  

Example.  Fetch models matching attributes, or annotate an existing catalog.  Simple example.

1.  Parse categories and attributes
2. Associate models
3. Export capturedroom


###  Parse attributes and categories to create folder hierarchy - 19:20
```swift
// Parse attributes and categories to create folder hierarchy

for category in CapturedRoom.Object.Category.allCases {

    let url = generateFolderURL(category: category, attributes: [])
    FileManager.default.createDirectory(at: url, withIntermediateDirectories: true)

    for attributes in category.supportedCombinations {
        let url = generateFolderURL(category: category, attributes: attributes)
        FileManager.default.createDirectory(at: url, withIntermediateDirectories: true)
    }
}
```

We iterate through all categories that roomplan supports.  Folder fo each category.  

For each category, we request attributes.  For each set of attributes, we create a folder.  We can add a 3d model corresponding to a category, set of attributes.

Now we need to create a catalog index.
###  Create a Catalog index - 20:00
```swift
// Create a Catalog index

struct RoomPlanCatalog: Codable {
    let categoryAttributes: [RoomPlanCatalogCategoryAttribute]
}

struct RoomPlanCatalogCategoryAttribute: Codable {
    enum CodingKeys: String, CodingKey {
        case folderRelativePath
        case category
        case attributes
        case modelFilename
    }
    let category: CapturedRoom.Object.Category
    let attributes: [any CapturedRoomAttribute]
    let folderRelativePath: String
    private(set) var modelFilename: String? = nil
    
    func encode(to encoder: Encoder) throws {
       …
    }
}
```

Now we save our index file as a list and our catalog as a bundle.
###  Create a Catalog bundle - 20:15
```swift
//  Create a Catalog bundle

let catalog = RoomPlanCatalog(categoryAttributes: categoryAttributes)
let plistEncoder = PropertyListEncoder()
let data = try plistEncoder.encode(catalog)
let catalogURL = inputURL.appending(path: "catalog.plist")
try data.write(to: catalogURL)
let fileWrapper = try FileWrapper(url: inputURL)
try fileWrapper.write(to: outputURL, options: [.atomic, .withNameUpdating],
                      originalContentsURL: nil)
```

Use the catalog bundle to…

Find the corresponding model.

Call the export function, specifying output url, model option, etc.

USDZ with 3d models that correspond to your scan.  Optionally take the usdz into blender, etc.

We added a prepopulated catalog in our sample code.


###  Instantiate a Model Provider from a Catalog - 20:22
```swift
// Instantiate a Model Provider from a Catalog

for categoryAttribute in catalog.categoryAttributes {
    guard let modelFilename = categoryAttribute.modelFilename else {
        continue 
    }
    let folderRelativePath = categoryAttribute.folderRelativePath
    let modelURL = url.appending(path: folderRelativePath).appending(path: modelFilename)
    if categoryAttribute.attributes.isEmpty {
       try modelProvider.setModelFileURL(modelURL, for: categoryAttribute.category)
    } else {
       try modelProvider.setModelFileURL(modelURL, for: categoryAttribute.attributes)
    }
}
```

###  Exporting a captured room to usdz with models - 20:47
```swift
// Exporting a captured room to usdz with models

try capturedRoom.export(to: outputURL, modelProvider: modelProvider, 
                        exportOptions: .model)
```

# Wrap up
* custom ARSession support
* New multi room experiences
* Accessibility improvements
* More accurate representation
* New workflows to customize roomplan output

