#tvOS 

New functionality in playback UI
# AVPlayerViewController
* Standard video playback experience on tvOS
* Wide variety of remotes
* Skipping, scanning, scrubbing
* Interstital support and more

[[Delivering intuitive media playback with AVKit - 19]]
# enhancements to playback UI

Redesigned UI.  Improved discoverability of common tasks like subtitles, etc.

* Link with tvOS 15 SDK
* Existing applicatons will get 'older' playback UI and functionality

# New APIs
## Title view
* Uses metadata embedded within media asset
* `commonIdentifierTitle`
* `iTunesMetadataTrackSubTitle`

Supplement custom metadata
* `AVPlayerItem.externalMetadata`
* Transport bar controls

## Transport bar controls
* Supported types
	* UIAction
	* UIMenu

```swift
let favoriteAction = UIAction(title: "Favorites", image: UIImage(systemName: "heart")) {
    // Add to favorites
}

let submenu = UIMenu(title: "Speed", options: [.displayInline, .singleSelection],
                     children: [ UIAction(…) ])

let menu = UIMenu(image: UIImage(systemName: "gearshape"), children: [submenu, UIAction(…)])
playerViewController.transportBarCustomMenuItems = [favoriteAction, menu]
```

[[Modernizing your UI for iOS 13]]

## Content tabs
depreacting `customInfoViewController`, now you have multiple controllers.

```swift
// Initialize content tab view controller

customViewController.preferredContentSize = CGSize(width: 0, height: 140)
customViewController.title = "Recommended"
```

System will size all VCs to the height of the tallest tab.  Size these consistently.

## Content configurations in TVUIKit

```swift
// Configure 16:9 UICollectionView cell
import TVUIKit

var contentConfiguration = TVMediaItemContentConfiguration.wideCell()
contentConfiguration.image = UIImage(imageLiteralResourceName: "tanu")
contentConfiguration.text = "Title"
contentConfiguration.secondaryText = "Secondary text"
contentConfiguration.badgeText = "NEW"
contentConfiguration.badgeProperties.backgroundColor = .systemRed
contentConfiguration.playbackProgress = 0.75

cell.contentConfiguration = contentConfiguration
```

For more information, see [[Modern cell configuration]]

## Contextual actions
Display relevant controls during playback.  `AVPlayerViewController.contextualActions`.

e.g. skip button during certain section.  `addPeriodicTimeObserver` `addBoundarytimeObserver`

This API is consistent with focus behavior and will maintain the now playing status of your application.
# Best practices
* Avoid adding extra gestures to playback UI
* Supplement missing metadata
* Enable picture in picture

[[Master picture-in-picture on tvOS]]

# Wrap up
* Build great experiences with standard playback UI
* New APIs in AVKit and TVUIKit

* https://developer.apple.com/documentation/avkit/customizing_the_tvos_playback_experience
* https://developer.apple.com/design/human-interface-guidelines/tvos/overview/themes/
* https://developer.apple.com/documentation/avkit

