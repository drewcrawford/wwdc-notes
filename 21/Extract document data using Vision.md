# Vision focus
* Image analysis
* Accessibility
* People
	* [[Detect people, faces, and poses using Vision]]
* Health
* Computational photography
	* Portrait mode, face detection, etc.
* Security
* Documents

# Documents
`VNRequests`
* Barcode detection
* Text recognition
* Contour detection
* Rectangle detection
* Document segmentation detection

# Barcode detection
New revision, `VNDetectBarcodesRequestRevision2`
`.codabar`
`.g1DataBar` and derivatives
`.microPDF417`
`microQR`

Small change in ROI behavior

When we do not specify a ROI, bbox is reported in relation to the full image.

When we specify an ROI, r2 now reports bounding box in relation to ROI.

Revison 1 always reports in relation to the full image.

When you compile your application and do not specify a revision, you get the latest revision.  Apps that specify revision 1 or do not recompile will get old behavior.
## Using the barcode detector
* Support for 1D and 2D codes
* Detect multiple codes at once
* Detect multiple symbologies at once
	* Takes longer

## How does the scanning work
* 1D codes get scanned as lines
	* Multiple detections for the same code
* 2D codes get scanned as units
	* One bounding box per code

Each barcode has its own observation
* 1D Codes can have multiple observations
Payload is the content of the barcode
* Use data detector for QR payloads - NSDataDetector

## Demo

```swift
import Foundation
import Vision

let url = URL(fileReferenceLiteralResourceName: "codeall_4.png") as CFURL

guard let imageSource = CGImageSourceCreateWithURL(url, nil),
      let barcodeImage = CGImageSourceCreateImageAtIndex(imageSource, 0, nil) else {
    fatalError("Unable to create barcode image.")
}

let imageRequestHandler = VNImageRequestHandler(cgImage: barcodeImage)

let detectBarcodesRequest = VNDetectBarcodesRequest()
detectBarcodesRequest.revision = VNDetectBarcodesRequestRevision2
detectBarcodesRequest.symbologies = [.codabar]

try imageRequestHandler.perform([detectBarcodesRequest])

if let detectedBarcodes = detectBarcodesRequest.results {

    drawBarcodes(detectedBarcodes, sourceImage: barcodeImage)
    
    detectedBarcodes.forEach {
        print($0.payloadStringValue ?? "")
    }
}




public func createCGPathForTopLeftCCWQuadrilateral(_ topLeft: CGPoint,
                                            _ bottomLeft: CGPoint,
                                            _ bottomRight: CGPoint,
                                            _ topRight: CGPoint,
                                            _ transform: CGAffineTransform) -> CGPath
{
    let path = CGMutablePath()
    path.move(to: topLeft, transform: transform)
    path.addLine(to: bottomLeft, transform: transform)
    path.addLine(to: bottomRight, transform: transform)
    path.addLine(to: topRight, transform: transform)
    path.addLine(to: topLeft, transform: transform)
    path.closeSubpath()
    return path
}


public func drawBarcodes(_ observations: [VNBarcodeObservation], sourceImage: CGImage) -> CGImage? {
    let size = CGSize(width: sourceImage.width, height: sourceImage.height)
    let imageSpaceTransform = CGAffineTransform(scaleX:size.width, y:size.height)
    let colorSpace = CGColorSpace.init(name: CGColorSpace.sRGB)
    let cgContext = CGContext.init(data: nil, width: Int(size.width), height: Int(size.height), bitsPerComponent: 8, bytesPerRow: 8 * 4 * Int(size.width), space: colorSpace!, bitmapInfo: CGImageAlphaInfo.premultipliedLast.rawValue)!
    cgContext.setStrokeColor(CGColor.init(srgbRed: 1.0,  green: 0.0,  blue: 0.0,  alpha: 0.7))
    cgContext.setLineWidth(25.0)
    cgContext.draw(sourceImage, in: CGRect(x: 0.0, y: 0.0, width: size.width, height: size.height))
    
    for currentObservation in observations {
        let path = createCGPathForTopLeftCCWQuadrilateral(currentObservation.topLeft,
                                                        currentObservation.bottomLeft,
                                                        currentObservation.bottomRight,
                                                        currentObservation.topRight,
                                                        imageSpaceTransform)
        cgContext.addPath(path)
        cgContext.strokePath()
    }
    return cgContext.makeImage()
}
```


# Text recognition
Introduced in 2019.  Fast vs accurate.

In the fast pass, we have a latin character recognizer.  Accurate pass uses an ML based recognizer

Each pass does language correction
Then recognized text.

Language selection affects first 2 phases.

## Best practices
* Query which languages are  supported `.supportedREcognitionLanguages`
* Order matters
	* First language may decide which model is used
* Think about your use case

## Demo

# Document detection
`VNDetectDocumentSegmentationRequest`
* ML-based detector
* Provides segmentation mask and corner points
* Realtime on devices with a Neural Engine
* Used in `VNDocumentCamera`

## Document segmentation vs Rectangle Detector
| `VNDetectDocumentSegmentationRequest`                                           | `VNDetectRectanglesRequest`                                       |
|---------------------------------------------------------------------------------|-------------------------------------------------------------------|
| Machine learning based running on NE (realtime), GPU (?), or CPU (not realtime) | Traditional algorithm running on CPU, realtime                    |
| Trained on documents, labels, signs, including non-rectangular shapes           | Detects edges and looks for intersections to form a quadrilateral |
| Provides segmentation Mask and corner points                                    | Corner pints only                                                 |
| Finds one document                                                              | N-rectangles including nested rectangles                          |

## Demo
```swift
import Foundation
import CoreImage
import Vision
import CoreML

guard var inputImage = CIImage(contentsOf: #fileLiteral(resourceName: "IMG_0001.HEIC"))
else { fatalError("image not found") }

inputImage

let requestHandler = VNImageRequestHandler(ciImage: inputImage)
let documentDetectionRequest = VNDetectDocumentSegmentationRequest()
try requestHandler.perform([documentDetectionRequest])

guard let document = documentDetectionRequest.results?.first,
      let documentImage = perspectiveCorrectedImage(from: inputImage, rectangleObservation: document) else {
          fatalError("Unable to get document image.")
      }
    
documentImage
let documentRequestHandler = VNImageRequestHandler(ciImage: documentImage)

/*
 TODO
  Detect barcodes
  Detect rectangles
  Recognize text
  Perform those requests
  Scan checkboxes
 */

var documentTitle = "Don't know yet"

let barcodesDetection = VNDetectBarcodesRequest() { request, _ in
    guard let result = request.results?.first as? VNBarcodeObservation,
          let payload = result.payloadStringValue else { return }
    documentTitle = "\(payload) was: "
}
barcodesDetection.symbologies = [.qr]

var checkBoxImages: [CIImage] = []
var rectangles: [VNRectangleObservation] = []

let rectanglesDetection = VNDetectRectanglesRequest { request, error in
    rectangles = request.results as! [VNRectangleObservation]
    // sort by vertical coordinates
    rectangles.sort{$0.boundingBox.origin.y > $1.boundingBox.origin.y}
    
    for rectangle in rectangles {
        guard let checkBoxImage = perspectiveCorrectedImage(from: documentImage, rectangleObservation: rectangle)
        else { print("Could not extract document"); return }
        checkBoxImages.append(checkBoxImage)
    }
}
rectanglesDetection.minimumSize = 0.1
rectanglesDetection.maximumObservations = 0

var textBlocks: [VNRecognizedTextObservation] = []

let ocrRequest = VNRecognizeTextRequest { request, error in
    textBlocks = request.results as! [VNRecognizedTextObservation]
}

do {
    try documentRequestHandler.perform([ocrRequest, rectanglesDetection, barcodesDetection])
} catch {
    print(error)
}


let classificationRequest = createclassificationRequest()

var index = 0
for checkBoxImage in checkBoxImages {
    let checkBoxRequestHandler = VNImageRequestHandler(ciImage: checkBoxImage)
    do {
        try checkBoxRequestHandler.perform([classificationRequest])
        if let classifications = classificationRequest.results as? [VNClassificationObservation] {
            if let topClassification = classifications.first
            {
                if topClassification.identifier == "Yes" && topClassification.confidence >= 0.9 {
                    for currentText in textBlocks {
                        if observationLinesUp(rectangles[index], with: currentText) {
                            let foundTextObservation = currentText.topCandidates(1)
                            documentTitle += foundTextObservation.first!.string + " "
                        }
                    }
                }
            }
        }
    } catch {
        print(error)
    }
    index += 1
}

print(documentTitle)



extension CGPoint {
    func scaled(to size: CGSize) -> CGPoint {
        return CGPoint(x: self.x * size.width, y: self.y * size.height)
    }
}
extension CGRect {
    func scaled(to size: CGSize) -> CGRect {
        return CGRect(
            x: self.origin.x * size.width,
            y: self.origin.y * size.height,
            width: self.size.width * size.width,
            height: self.size.height * size.height
        )
    }
}

public func observationLinesUp(_ observation: VNRectangleObservation, with textObservation: VNRecognizedTextObservation ) -> Bool {
    // calculate center
    let midPoint =  CGPoint(x:textObservation.boundingBox.midX, y:observation.boundingBox.midY)
    return textObservation.boundingBox.contains(midPoint)
}

public func perspectiveCorrectedImage(from inputImage: CIImage, rectangleObservation: VNRectangleObservation ) -> CIImage? {
    let imageSize = inputImage.extent.size
    
    // Verify detected rectangle is valid.
    let boundingBox = rectangleObservation.boundingBox.scaled(to: imageSize)
    guard inputImage.extent.contains(boundingBox)
    else { print("invalid detected rectangle"); return nil}
    
    // Rectify the detected image and reduce it to inverted grayscale for applying model.
    let topLeft = rectangleObservation.topLeft.scaled(to: imageSize)
    let topRight = rectangleObservation.topRight.scaled(to: imageSize)
    let bottomLeft = rectangleObservation.bottomLeft.scaled(to: imageSize)
    let bottomRight = rectangleObservation.bottomRight.scaled(to: imageSize)
    let correctedImage = inputImage
        .cropped(to: boundingBox)
        .applyingFilter("CIPerspectiveCorrection", parameters: [
            "inputTopLeft": CIVector(cgPoint: topLeft),
            "inputTopRight": CIVector(cgPoint: topRight),
            "inputBottomLeft": CIVector(cgPoint: bottomLeft),
            "inputBottomRight": CIVector(cgPoint: bottomRight)
        ])
    return correctedImage
}

public func createclassificationRequest() -> VNCoreMLRequest
{
    let classificationRequest: VNCoreMLRequest = {
        // Load the ML model through its generated class and create a Vision request for it.
        do {
            let coreMLModel = try MLModel(contentsOf: #fileLiteral(resourceName: "CheckboxClassifier.mlmodelc"))
            let model = try VNCoreMLModel(for: coreMLModel)
            
            return VNCoreMLRequest(model: model)
        } catch {
            fatalError("can't load Vision ML model: \(error)")
        }
    }()
    return classificationRequest
}
```

By default, rectangle detector only looks for rectangles that are >= 20% of the image.  Set `.minimumSize` to the appropriate value.

By default, we only return 1 rectangle.  Set `.maximumObservations = 0`.

# Wrap up
* document analysis is a focus in Vision
* Barcode detection is more versatile than a barcode scanner
* New document segmentation detection

[[Explore Computer Vision APIs]]
[[Text recognition in Vision Framework - 19]]

https://developer.apple.com/documentation/vision

