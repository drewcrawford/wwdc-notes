# What is live text?
* Text interactions
* Translate
* Data detection
* QR codes

# Live Text API
Available in Swift.  Wroks with static or still iamges, and can be adapted for video frames.

If you need to analyze video, VisionKit has a datascanner available

[[Capture machine-readable codes and text with VisionKit]]

Available
iOS/iPadOS 16+ when 2018 or newer with neural engine
All devices on macOS 13+

ImageAnalyzer => ImageAnalysis

 
# Adopting live text
Hector's cafe and diner.  Notice I can zoom and pan.  But try as I might, I can't select any text orany data detectors.

To add live text, I'll be modifying VC subclass.  First, I need an ImageNalayzer and an ImageAnalysisInteraction.

```swift
import UIKit
import VisionKit

class LiveTextDemoController: BaseController, ImageAnalysisInteractionDelegate, UIGestureRecognizerDelegate {
   
    let analyzer = ImageAnalyzer()
    let interaction = ImageAnalysisInteraction()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        imageview.addInteraction(interaction)
    }
    
    override var image: UIImage? {
        didSet {
            interaction.preferredInteractionTypes = []
            interaction.analysis = nil
            analyzeCurrentImage()
        }
    }
    
    func analyzeCurrentImage() {
        if let image = image {
            Task {
               let configuration = ImageAnalyzer.Configuration([.text, .machineReadableCode])
                do {
                    let analysis = try await analyzer.analyze(image, configuration: configuration)
                    if let analysis = analysis, image == self.image {
                        interaction.analysis = analysis;
                        interaction.preferredInteractionTypes = .automatic
                    }
                }
                catch {
                    // Handle error…
                }
            }
        }
    }
}
```


# Tips and tricks
Most developers want automatic, which provides text selection but also data detectors.  Detect any applicable detected items, and allows one-tap access.

If it maeks sense for your app to have only text selection, you may set `.textSelection`.

If it makes sense to only have data detectors, set type to that.  In this mode, since selection is disabled, you will not see a live text button.  

empty set disables
can still long press for data detectors.  See `.allowLongpressForDataDetectorsInTextMode`.

## Supplementary interface
bottom buttons
* live text ubtton
* quick actions
	* any data detectors from the analysis.
	* size, position, visibility is automatic
	* Customizable

`.isSupplementaryInterfaceHidden` => hides completely

Content insets.  `supplementaryInterfaceContentInsets` to inset them inside.

`supplementaryInterfaceFont` to get a custom font.
Note that for button sizing consistency, live text will ignore the point size.

## match highlights to content
With UIImageView, visionkit can use content mode to aclculate contents rect automatically.  Here, interaction's view has bounds bigger than image content, but i susing default content rect which is a unit rect.

`contentsRect(for interaction:)` and return a rectangle describing how image content relates to bounds.  This corrects this.

`.setContentsRectNeedsUpdate()` to change when bounds has not changed.  

## Interaction placement
* over image content
* UIImageView handles this for you
	* Just set the interaction on the imageview
* If your image view is located inside the scrollview.  You may be tempted to place on the scrollview.  Not recommended as it has continually changing content rect.
	* Place interaction on the view, even if it's inside the scrollview with magnification enabled
## Handling gesture conflicts
It's possible the itneraction responds to gestures and events your apps hould handle or vice versa.

Implement delegate method `interaction(shouldBeginAt point for interactionType)`.  Maybe check if interaction has an interactive item or active text selection (tap off text to deselect?)

On the other hand, maybe your app needs to implement gestureRecognizerShouldBegin.  Here, I check the interactiveItem or active text selection and say my GR should not begin in that case.

In a scrollivew with magnification applied, consider the pattern.
1.. Convert GR(location in: nil) (aka, window)
2.  Convert that to the interaction view coordinates


Maybe override `hitTest`.  Similar checks, here we return the appropriate view.

## Maximize performance
* Use one ImageAnalyzer in your app
* Minimize image conversions
	* If you do happen to have a CVPixelBuffer, that would be most efficient
* Analyze only when image appears on screen.
* Wait for scrollviews to stop scrolling.

## Other live text availability
* UITextField
* UITextView
* WebKit
* Quick Look
[[Use the camera for keyboard input in your app]]
[[Quick Look previews from the ground up - 18]]

## AVKit
#avkit 
AVPlayerView
AVPlayerViewController
`allowsVideoFrameAnalysis`
Enabled by default
only non-fairplay content

### AVPlayerLayer
You are reponsible for managing analysis and interaction
must get frame after video pauses =>  `.currentlyDisplayedPixelBuffer()`

Will only return avalue if video is paused
Do not write to resulting pixel buffer!!
Non-fairplay protected content only



* https://developer.apple.com/forums/tags/wwdc2022-10026
* https://developer.apple.com/forums/create/question?&tag1=352030&tag2=467030
* https://developer.apple.com/documentation/visionkit/enabling_live_text_interactions_with_images
