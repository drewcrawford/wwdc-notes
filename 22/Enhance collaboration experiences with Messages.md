# meet messages collaboration
iOS 16, macos Ventura.  Tie a document to conversations by sharing via messages.
Activity is shown in messages conversations and facetime calls.

Manage details of the collaboration and messages conversation.  Buidls on technologies in the share sheet and drag-and-drop.
# Prepare for collaboration
* CloudKit
* iCloud Drive
* Custom infrastructure
[[Integrate your custom collaboration app with Messages]]

## CloudKit
Create collaboration object
Identify colalboration UI
Update to new API

We will deprecate the existing appkit API.

Transport data to other processes via `NSItemProvider`
CKShare or preparation handler
CKContainer
CKAllowedSharingOptions => access and permissions

```swift
// CloudKit collaboration object

// Starting collaboration
let itemProvider = NSItemProvider()
itemProvider.registerCKShare(container: container, 
                             allowedSharingOptions: CKAllowedSharingOptions.standard, 
                             preparationHandler: {
    // Create your share and save to server, or throw error
    return savedShare
})

// Inviting to existing collaboration
let itemProvider = NSItemProvider()
itemProvider.registerCKShare(share, 
                             container: container, 
                             allowedSharingOptions: CKAllowedSharingOptions.standard)
```

* Provides free iCloud database
* built in data sharing method
[[What's new in CloudKit]]

## iCloud Drive
* Provide file URL
* Identify colalboration UI
* Prepare to replace

## Custom
`SWCollaborationMetadata`
Update collaboration UI


# Initiate collaboration
Two different ways through share sheet, and drag-and-drop to messages (your app or files/Finder).

Give the share sheet the collaboration object you prepared earlier.  In macOS this is shown in a poppover with title and image, plus a row of conversation suggestions.

[[22/What's new in AppKit]]

## Share sheet and share popover
`UIActivityViewController` => iOS
`NSSharingServicePicker` => macOS

```swift
// Setting up Share Sheet - iOS and Mac Catalyst

let activityViewController = UIActivityViewController(activityItems: [collaborationObject], applicationActivities: nil)

presentingViewController.present(activityViewController, animated: true)
```

```swift
// Setting up Share Popover - macOS

let sharingServicePicker = NSSharingServicePicker(items: [collaborationObject])

sharingServicePicker.show(relativeTo: view.bounds, of: view, preferredEdge: .minY)
```

Provide title and image for headers.
`UIActivityItemsConfiguration` or `UIActivityItemSource` to provide `LPLinkMetadata`.

```swift
// Providing CloudKit metadata - iOS

let configuration = UIActivityItemsConfiguration(itemProviders: [collaborationItemProvider])
configuration.perItemMetadataProvider = { (_, key) in
    switch key {
    case .linkPresentationMetadata:
        // Create LPLinkMetadata with title and imageProvider
        return metadata
    default:
        return nil
    }
}

let activityViewController = UIActivityViewController(activityItemsConfiguration: configuration)
```
On macOS, `NSPreviewREpresentingActivityItem`, see docs.

```swift
// Providing CloudKit metadata - macOS

let title = “Shared Item”
let image = NSImage(contentsOfFile: “Shared_Item_Preview_Image.png”)
let icon = NSImage(contentsOfFile: “App_Icon.png”) // Shared item source

let previewRepresentingItem = NSPreviewRepresentingActivityItem(item: collaborationItemProvider, 
                                                                title: title, 
                                                                image: image, 
                                                                icon: icon)

let picker = NSSharingServicePicker(items: [previewRepresentingItem])
```

SwiftUI, provide `Transferable` item to share
`CKShareTransferableRepresentation`

[[Meet Transferable]]
[[22/What's new in SwiftUI]]

```swift
// SwiftUI CloudKit Transferable

struct Note: Transferable {
    // Properties of the note e.g. name, preview image, content, ID, …
    var share: CKShare?
    func saveCKShareToServer() async throws -> CKShare { … }

    static var transferRepresentation: some TransferRepresentation {
        CKShareTransferRepresentation { note in
            if let share = note.share {
                return .existing(share, container: container, options: options)
            } else {
                return .prepareShare(container: container, options: options) {
                    return try await note.saveCKShareToServer()
                }
            }
        }
    }
}
```

SwiftUI ShareLink - iCloud drive and custom.
* iCloud drive
	* file URL
* Custom infrastructure
	* SWCollaborationMetadata

```swift
// SwiftUI ShareLink adoption

struct ContentView: View {
    @State let item = ShareItem()

    var body: some View {
        ShareLink(item: item, preview: SharePreview(item.title, image: item.previewImage))
    }
}
```

## Access and permissions
CloudKit
* registered `CKAllowedSharingOptions`

iCloud drive
* standard set
Custom infrastructure
* custom options

## drag and drop
drag document into messages and get new collaboration-enabled rich link.

Can select colalboration options.


# Collaboration UI in your app
Further integrate UI into your app.

Collaboration button is placed in your app's navigation and shows the group photo in the associated messages group.  Can also show when others are present in the document.

When you tap the collaboration button, the new popover appears.  Customizable popover shows overview of the collaboration, allows users to initiate communication via messages and facetime.

* SWCollaborationView
	* Initialize with NSItemProvider
	* Show custom content
	* Active participant count (iPad/Mac)
	* Custom manage button title

```swift
// Collaboration View

//has items we discussed previously, one of 3 cases
let collaborationView = SWCollaborationView(itemProvider: itemProvider)

collaborationView.activeParticipantCount = myModel.activePeople.count

collaborationView.contentView = MyView(model: myModel)

collaborationView.manageButtonTitle = "Custom Manage Button"
```

contentView makes this customizable.  Add your own content.  ex, in pages, the content view contains current participants and their cursor view options.

For iCloud/CloudKit, the manage button brings up the manage UI.  More on this shortly.

```swift
// Collaboration View

let collaborationView = SWCollaborationView(itemProvider: itemProvider)

collaborationView.activeParticipantCount = myModel.activePeople.count

collaborationView.contentView = MyView(model: myModel)

collaborationView.manageButtonTitle = "Custom Manage Button"
```

Include your own manage button in the content view for custom integration, *one will not be provided for you*.  

Use collaborationView as the custom view of a `UIBarButtonItem`.

CloudKit/iCloud drive are provided a button in collaboration popover.
* system-provided manage UI
* observe changes using delegate protocols
	* `UICloudSharingControllerDelegate` and NS etc.

# collaboration updates
* `CKSystemSharingUIObserver` (cloudkit)
* callbacks when sharing is started/stopped
* see docs

```swift
// Observing CKShare Changes

let observer = CKSystemSharingUIObserver(container: container)

observer.systemSharingUIDidSaveShareBlock = { _, result in
    switch result {
    case .success(let share):
        // Handle successfully starting share
    case .failure(let error):
        // Handle error
    }
}
```

similar for stopping sharing.

We've added APIS to enable you to post notices summarizing updates to collaboration at the top of the relevant messages convo.  To post a notice
`SWCollaborationHighlight`
`SWHighlightCenter`

Learn more about this in [[Add shared with you to your app]]

## Notices in messages
* cloudkit
* custom collaboration infrastructures => view the [[Integrate your custom collaboration app with Messages]] for details

Events initialized with `SWHighlight`
event types:
* change
* mention
* persistence
* membership

```swift
// Post an SWHighlightChangeEvent Notice

let highlightCenter: SWHighlightCenter = self.highlightCenter

let highlight = try highlightCenter.collaborationHighlight(forURL: ckShareURL, error: &error)

let editEvent = SWHighlightChangeEvent(highlight: highlight, trigger: .edit)

highlightCenter.postNotice(for: editEvent)
```

```swift
// Post an SWHighlightMembershipEvent Notice

let highlightCenter: SWHighlightCenter = self.highlightCenter

let highlight = try highlightCenter.collaborationHighlight(forURL: ckShareURL, error: &error)

let membershipEvent = SWHighlightMembershipEvent(highlight: highlight, 
                                           trigger: .addedCollaborator)

highlightCenter.postNotice(for: membershipEvent)
```

Group membership parity
Keep documents in sync when membership changes.  For CloudKit and iCloud drive, we do the work.  

When someone new is added to the group, you can add them to the share.  Similar when removed.

* adopt `SWCollaborationActionHandler`

# next steps
* Prepare for collaboration
* Integrate collaboration UI
* Adopt notices
