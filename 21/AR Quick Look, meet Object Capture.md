AR quicklook is a builtin systemwide AR viewer.  Safari, messages, files, and more.

Embed 3d models letting people interact with them.  Showcase your product, promote an event, or provide additional content in an immersive experience.

# Creating 3D content
If you wanted to create 3D content, it required teh use of 3d modeling software.  But they're usually expensive and can be difficult to approach.

This year, we added object capture.  Create a high-quality 3D model using still images.  Does heavy lifting, to create  USDZ file.

Use reality composer to add interactive custom behaviors, e.g. tap triggers, camera actions, etc.

These technologies make it easier than ever for anyone to create an immersive AR experience.

> Object capture does not make your plants grow automatically.

Repeated these steps for each plant to generate separate usdz models.  Swap plants with behaviors.


# Content best practices
Reduced, medium, full/raw.  See some other talk where I wrote this down.
Mesh levels depend on detail settings.  Reduced/medium offer a great balance between visual fidelity and filesize.  
We recommend you export in reduced and medium detail settings.
## Reduced
As seen, reduced detail setting will generate models with smallest size.  Ideal for web distribution.  Recommended setting for combining multiple assets in a scene.
Great default choice.
## Medium
Complex objects
Single hero asset
Suitable for apps
Evaluate both reduced and medium settings
Test on variety of devices
Ensure quality source images
* Take sharp photos
* Maintain >70% overlap between photos

[[Capturing photographs for RealityKit object creation]]
[[Create 3D models with Object Capture]]


# Integrate AR Quick Look
Possible to embed with a few lines of code.
```swift
// File: MyPreviewController.swift
func presentARQuickLook() {
	let previewController = QLPreviewController()
	previewController.dataSource = self
	present(previewController, animated: true)
}

// MARK: QLPreviewControllerDataSource
func previewController(
  _ controller: QLPreviewController, previewItemAt index: Int) -> QLPreviewItem {
	  let previewItem = ARQuickLookPreviewItem(fileAt: fileURL) // Local file URL

	  return previewItem
}
```

Website use `usdz`, `rel="ar"`

Possible to disable content scaling here as well.  `allowsContentScaling=0`.

* Easy to add AR experience to your app or website
* Customizable AR quick look behavior
* Integrates with apple pay and custom actions

[[Shop online with AR Quick Look]]
[[Advances in AR Quick Look - 19]]
# Real-world applications
* RealityKit
* usdz
* AR quicklook

## Ecommerce
* 3d models help preview in your environment
* Easily create 3d models of products
* Swap between products with Reality Composer

## Museums
* Artifacts need to be preserved in cases
* Tour a museum from anywhere
* Create new forms of exhibits
* Add audio for narration and increased accessibility

## Education
Diagrams and videos can be 3D
New way to teach 3d concepts
Teachers can create engaging lessons
Interactivity helps students learn at own pace
Kids can express creativity in AR.  ex, clothes app allows kids to create 3d models of their favorite tool.

# Wrap up
* Develop the next new immersive AR experience
* Create 3D content for AR Quick Look with Object Capture
* Create multi-asset scenes and add interactivity with Reality Composer

[[Create 3D models with Object Capture]]

* https://developer.apple.com/documentation/arkit/adding_an_apple_pay_button_or_a_custom_action_in_ar_quick_look
* https://developer.apple.com/arkit/gallery
* https://developer.apple.com/documentation/arkit


