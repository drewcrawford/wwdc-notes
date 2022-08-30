#sharedwithyou 
# Prerequisites
* existing collaboration ifnrastructure
* universal link adoption

[[Add Shared with You to your app]]
[[Enhance collaboration experiences with Messages]]


# Collaboration lifecycle
When a user decides to share a collaboration,

1.  Create metadata to represent the content.  Includes sahre options the user can configure, and a number of other properties.
2. Share sheet/drag&drop.  Allows draft to bes taged in the compose field.
3. Must be presented by a universal link.  best to defer this until just before messages is sent.  Mayt depend on options or recipients.  User chooses these and sends.
4. Asks your app for the universal link and a device-independent identifier.  Messages provides a set of cryptographic identities representing the recipients.  App will use these identities later.
5. App stores those identities on its servers and associates them with the shared content.  The message is sent to the recipients.  
6. On receiving device, your app receives the universal link `openURL`.  When your app detects that you don't yet have access to the document, it queries for proof of identity
7. sends the signed identity proof to your server.  If signature is valid, it compares proof against identities from the sending device.  If it maches, server grants access to the account.

# Starting a collaboration
* Title
* Local lidentifier
* Initiator name and account handle
* Default share options
```swift
let localIdentifier = SWLocalCollaborationIdentifier(rawValue: "identifier")
let metadata = SWCollaborationMetadata(localIdentifier: localIdentifier)
metadata.title = "Content Title"
metadata.initiatorHandle = "user@example.com"

let formatter = PersonNameComponentsFormatter()
if let components = formatter.personNameComponents(from: "Devin") {
    metadata.initiatorNameComponents = components
}

metadata.defaultShareOptions = ...
```

Share options.  You use a few classes to define ioptions, starting with SWCollaborationOption
* switch, or mutually exclusive settings
* title and identifier
* selected or unselected

* SWCollaborationOptionsGroup
	* individuals witches
* SWCollaborationOptionsPickerGroup
	* Mutually exclusive values for setting

SWCollaborationShareOptions
* contains option groups for metadata's `defaultShareOptions`
* Summary string

```swift
let permission = SWCollaborationOptionsPickerGroup(identifier: UUID().uuidString, 
                                                   options: [
    SWCollaborationOption(title: "Can make changes", identifier: UUID().uuidString),
    SWCollaborationOption(title: "Read only", identifier: UUID().uuidString)
])
permission.options[0].isSelected = true
permission.title = "Permission"

let additionalOptions = SWCollaborationOptionsGroup(identifier: UUID().uuidString, 
                                                    options: [
    SWCollaborationOption(title: "Allow mentions", identifier: UUID().uuidString),
    SWCollaborationOption(title: "Allow comments", identifier: UUID().uuidString)
])
additionalOptions.title = "Additional Settings"
let optionsGroups = [permission, additionalOptions]
metadata.defaultShareOptions = SWCollaborationShareOptions(optionsGroups: optionsGroups)
```

[[Meet Transferable]]
[[22/What's new in swiftUI]]
```swift
struct CustomCollaboration: Transferable {
    var name: String

    static var transferRepresentation: some TransferRepresentation {
        ProxyRepresentation { customCollaboration in
            SWCollaborationMetadata(
                localIdentifier: .init(rawValue: "com.example.customcollaboration"),
                title: customCollaboration.name,
                defaultShareOptions: nil,
                initiatorHandle: "johnappleseed@apple.com",
                initiatorNameComponents: nil
            )
        }
    }
}
```

```swift
struct ContentView: View {
    var body: some View {
        ShareLink(item: CustomCollaboration(name: "Example"), preview: .init("Example"))
    }
}
```

For UIKit and appkit apps, use NSItemProvider
SWCollaborationMetadata conforms to NSItemProviderReeading, etc.  
Register 
* collaboration metadata
* other content representations

Use on iOS and iPadOS
* UIActivityVC
* UIGragItem
macOS
* NSSharingServicePicker

```swift
func presentActivityViewController(metadata: SWCollaborationMetadata) {
    let itemProvider = NSItemProvider()
    itemProvider.registerObject(metadata, visibility: .all)
    let activityConfig = UIActivityItemsConfiguration(itemProviders: [itemProvider])
    let shareSheet = UIActivityViewController(activityItemsConfiguration: activityConfig)
    present(shareSheet, animated: true)
}
```

drag and drop:

```swift
func createDragItem(metadata: SWCollaborationMetadata) -> UIDragItem {
    let itemProvider = NSItemProvider()
    itemProvider.registerObject(metadata, visibility: .all)
    return UIDragItem(itemProvider: itemProvider)
}
```

macOS sharing popover

```swift
func showSharingServicePicker(view: NSView, metadata: SWCollaborationMetadata) {
    let itemProvider = NSItemProvider()
    itemProvider.registerObject(metadata, visibility: .all)
    let picker = NSSharingServicePicker(items: [itemProvider])
    picker.show(relativeTo: view.bounds, of: view, preferredEdge: .minY)
}
```

macOS drag and drop NSPasteboardItem extension

```swift
func createPasteboardItem(metadata: SWCollaborationMetadata) -> NSPasteboardItem {
    let pasteboardItem = NSPasteboardItem()
    pasteboardItem.collaborationMetadata = metadata
    return pasteboardItem
}
```

SWCollaborationCoordinator
* singleton
* actionhandler delegate
* Register delegate upon app launch
* Handle actions immediately

will be launched in the background if needed?
```swift
private let collaborationCoordinator = SWCollaborationCoordinator.shared

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]?) -> Bool {
    // Conform to the SWCollaborationActionHandler protocol
    collaborationCoordinator.actionHandler = self
}
```

SWAction.
* represents work for your pap
* fulfill when complete
* fail otherwise

SWStartCollaborationAction
* SWCollaborationmetadata with selected options
* fulfill with universal link and identifier
* failing cancels the message

```swift
func collaborationCoordinator(_ coordinator: SWCollaborationCoordinator, 
                              handle action: SWStartCollaborationAction) {
    let localID = action.collaborationMetadata.localIdentifier.rawValue
    let selectedOptions = action.collaborationMetadata.userSelectedShareOptions
    let prepareRequest = APIRequest.PrepareCollaboration(localID: localID, selectedOptions)
    Task {
        do {            
            let response = try await apiController.send(request: prepareRequest)
            let identifier = response.deviceIndependentIdentifier
            action.fulfill(using: response.url, collaborationIdentifier: identifier)
        } catch {
            Log.error("Caught error while preparing the collaboration: \(error)")
            action.fail() // cancels the message
        }
    }
}
```

* SWUpdateCollaborationParticipantsAction
	* crypto identities of participants
	* identities derviced from collaboration identifier
	* store identities on sever for later verification
	* fulfill the action to send the message

```swift
func collaborationCoordinator(_ coordinator: SWCollaborationCoordinator, 
                              handle action: SWUpdateCollaborationParticipantsAction) {
    let identifier = action.collaborationMetadata.collaborationIdentifier
    let participants: [Data] = action.addedIdentities.compactMap { $0.rootHash }
    let addParticipants = APIRequest.AddParticipants(identifier: identifier, participants)
    Task {
        do {            
            try await apiController.send(request: addParticipants)
            action.fulfill() // sends the URL provided by the start action
        } catch {
            Log.error("Caught error while adding participants to collaboration: \(error)")
            action.fail() // cancels the message 
        }
    }
}
```

Each identity has a property called a `rootHash`.  This is what we store on the server for future use.

# Verifying access
rootHash is used for this verification.  Secure value used to uniquely identify a particpant on their devices.  To verify, understand how to compute this.

1.  message is sent to each device.  Messages has a cryptographic public key.
2. Root has h is derived from the set of public keys registered to each recipient
3. It's the roothash of a datastructure called a merkle tree.  Binary tree that is filled by performing a sequence of hashing operations.
4. Keys are used as the leaves of this tree.  hashing algorithm ensures that the root node can only be computed from that set of keys.

ex the user has 3 devices and public keys.  Using a process called key diversification.  Set is padded with random keys up to a fixed value.
Leaf nodes are created by hashing each key.  Here we use sha256.
Each pair of leaf notes are concatenated and hashed to derive the parent nodes.
This process is repeated with the parent nodes, and repeated again until we have a single root node.

It's possible to generate a root hash for a subset of nodes, e.g. intermediate hashes.  The subset you need is called a "proof of inclusion".

## SWCollaborationHighlight
* subclass of SWHighlight
	* Retrieve from sWhighlightCenter
	* Used to retrieve proof of inclusion

## Signed identity proof
Use SWHighlightCenter to retrieve
* SWCollaborationHighlight
* SWPersonIdentityProof
* Signed data

```swift
func application(_ app: UIApplication, open url: URL, 
               options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    let highlightCenter: SWHighlightCenter = self.highlightCenter
    let challengeRequest = APIRequest.GetChallengeData()
    Task {
        do {
            let highlight = try highlightCenter.collaborationHighlight(for: url)
            let challenge = try await apiController.send(request: challengeRequest)
            let proof = try await highlightCenter.getSignedIdentityProof(for: highlight, 
                                                                       using: challenge.data)
    let proofOfInclusionRequest = APIRequest.SubmitProofOfInclusion(for: proof)
            let result = try await apiController.send(request: proofOfInclusionRequest)
            documentController.update(currentDocument, with: result)
        } catch {
            Log.error("Caught error while generating proof of inclusion: \(error)")
        }
    }
}
```

## Signed data
The signature is an elliptic curve digital signature algorithm (ECDSA)
Signature of the given data over the P-256 elliptic curve
Using SHA-256 as a hash function

> You can do this with most commonly-used encryption libraries

Use the identity proof to recompute the root hash.

example implementation

```swift
func generateRootHashFromArray(localHash: SHA256Digest, inclusionHashes: [SHA256Digest], 
                       publicKeyIndex: Int) -> SHA256Digest {
    guard let firstHash = inclusionHashes.first else { return localHash }
    // Check if the node is the left or the right child
    let isLeft = publicKeyIndex.isMultiple(of: 2)
    // Calculate the combined hash
    var rootHash: SHA256Digest
    if isLeft {
        rootHash = hash(concatenate([localHash, firstHash]), using: .sha256)
    } else {
        rootHash = hash(concatenate([firstHash, localHash]), using: .sha256)
    }
    // Recursively pass in elements and move up the Merkle tree
    let newInclusionHashes = inclusionHashes.dropFirst()
    rootHash = generateRootHashFromArray(
        localHash: rootHash,
        inclusionHashes: Array(newInclusionHashes),
        publicKeyIndex: (publicKeyIndex / 2)
    )
    return rootHash
}
```

Server checks that the root hash is in the list of root hashes the owner documented during sending.  Hash is present, so the server can grant access.  grant access to the document with coconfidence

## Steps
* look up the SWcollaborationHighlight for URL
* Sign data for server and retrieve proof of inclusion
* send signed data and proof to serve
* rverify siganture
* generate root hash using proof of inclusion
* compare root hash to known identities

# Participant changes
A user can propagate changes to your app.  In this scenario, 
* sWUpdateCollaborationParticipantsAction
* contains added/removed SWPersonIdentities
* Add: same flow as verifying access
* Remove: look up account for removed identity

```swift
func collaborationCoordinator(_ coordinator: SWCollaborationCoordinator, 
                              handle action: SWUpdateCollaborationParticipantsAction) {
    // Example of removing participants only. Handle the added identities here too.
    let identifier = action.collaborationMetadata.collaborationIdentifier
    let removed: [Data] = action.removedIdentities.compactMap { $0.rootHash }
    let removeParticipants = APIRequest.RemoveParticipants(identifier: identifier, removed)
    Task {
        do {            
            try await apiController.send(request: removeParticipants)
            action.fulfill()
        } catch {
            log.error("Caught error while adding participants to collaboration: \(error)")
            action.fail()
        }
    }
}
```
# Posting notices
Your app posts notices to be shown directly in messages.  There are a few types of supported notices.

Displayed as a banner right in the conversation where the link was shared.  Includes a description of what changed and who made the change.

## SWHighlightEvent protocol
Events intiialized with `SWHighlight`
Event types:
* change event
* membership event
* mention event
* persistence event

change event
```swift
func postContentEditEvent(identifier: SWCollaborationIdentifier) throws {
    let highlightCenter: SWHighlightCenter = self.highlightCenter
    let highlight = try highlightCenter.collaborationHighlight(forIdentifier: identifier)

    let editEvent = SWHighlightChangeEvent(highlight: highlight, trigger: .edit)

    highlightCenter.postNotice(for: editEvent)
}
```

Post an SWHighlightMentionEvent notice

```swift
func postContentEditEvent(identifier: SWCollaborationIdentifier) throws {
    let highlightCenter: SWHighlightCenter = self.highlightCenter
    let highlight = try highlightCenter.collaborationHighlight(forIdentifier: identifier)

    let editEvent = SWHighlightChangeEvent(highlight: highlight, trigger: .edit)

    highlightCenter.postNotice(for: editEvent)
}
```

```swift
func postMentionEvent(identifier: SWCollaborationIdentifier, mentionedRootHash: Data) throws {
    let mentionedIdentity = SWPerson.Identity(rootHash: mentionedRootHash)

    let highlightCenter: SWHighlightCenter = self.highlightCenter
    let highlight = try highlightCenter.collaborationHighlight(forIdentifier: identifier)

    let mentionEvent = SWHighlightMentionEvent(highlight: highlight,
                                               mentionedPersonIdentity: mentionedIdentity)
    highlightCenter.postNotice(for: mentionEvent)
}
```

```swift
func postContentRenamedEvent(identifier: SWCollaborationIdentifier) throws {
    let highlightCenter: SWHighlightCenter = self.highlightCenter
    let highlight = try highlightCenter.collaborationHighlight(forIdentifier: identifier)

    let renamedEvent = SWHighlightPersistenceEvent(highlight: highlight, trigger: .renamed)
    highlightCenter.postNotice(for: renamedEvent)
}
```

# Next steps
* set up collaboration
* verify access
* Keep track of participant changes
* Post notices



* https://developer.apple.com/forums/tags/wwdc2022-10093
* https://developer.apple.com/forums/create/question?&tag1=163&tag2=359030

