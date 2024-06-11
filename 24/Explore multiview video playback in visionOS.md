Learn how AVExperienceController can enable playback of multiple videos on Apple Vision Pro. Review best practices for adoption and explore great use cases, like viewing a sports broadcast from different angles or watching multiple games simultaneously. And discover how to design a compelling and intuitive multiview experience in your app.

###  7:47 - Supply a custom browser view controller
```swift
import AVKit
AVMultiViewManager.default.contentSelectionViewController = multiViewController()
```

###  8:09 - Add content to multiview
```swift
import AVKit
let controller = AVPlayerViewController()
let experienceController = controller.experienceController
experienceController.allowedExperiences = .recommended(including: [.multiView])
await experienceController.transition(to: .multiView)
```

###  8:47 - Remove content from multiview
```swift
import AVKit
let experienceController = …
await experienceController.transition(to: .embedded)
```

