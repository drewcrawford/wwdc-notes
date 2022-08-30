#pdfkit

# PDFKit Review
View, edit, and write PDF files
iOS, macOS, and mac catalyst
swiftUI using UIViewRepresentable
Four core classes:
* PDFView
* PDFDocument
	* PDFPage
		* PDFAnnotation

# Live text and forms
Now supports live text.  Different than photos where text is small you tap to copy.
WithPDF => it behaves like text.
* text selection and search
* Processed on demand
* Processed in place
* Ability to save

Forms:
* Suports document without built-in text fields
* Tap through text fields and enter text

# Create PDFs from images
`init(:image: CGImage, andOptions options:)`...

Options:
* mediaBox => size of the page.  Image exactly, or paper size
* rotation => portrait or landscape
* upscaleIfSmaller => image can be upscaled
# Overlay views
In the past, the only way to do addtiional drawing on PDFs was to subclass or used custom annotations.  But in iOS16, it's possible to overlay your view on top of each PDF page.  Live, fully interactive views that appear on top of PDF pages.

1.  Install your overlay view
2. Save your content in the PDF
3. Best practices when saving

PDFs can contain a large number of pages => don't create view for each page.  
PDFKit requests you roverlay views at the right time.
`PDFPageOverlayViewProvider`.  
Note that `PDFKitPlatformView` is just NSView or UIView as appropriate.

Can use the `willEndDisplaying` to set aside its view data.


```swift
class Coordinator: NSObject, PDFPageOverlayViewProvider {

    var pageToViewMapping = [PDFPage: UIView]()
   
    func pdfView(_ view: PDFView, overlayViewFor page: PDFPage) -> UIView? {
        var resultView: PKCanvasView? = nil
        
        if let overlayView = pageToViewMapping[page] {
            resultView = overlayView
        } else {
            let canvasView = PKCanvasView(frame: .zero)
            canvasView.drawingPolicy = .anyInput
            canvasView.tool = PKInkingTool(.pen, color: .systemYellow, width: 20)
            canvasView.backgroundColor = UIColor.clear
            pageToViewMapping[page] = canvasView
            resultView = canvasView
        }
        
        // If we have stored a drawing on the page, set it on the canvas
        let page = page as! MyPDFPage
        if let drawing = page.drawing {
            resultView?.drawing = drawing;
        }
        return resultView
    }

    func pdfView(_ pdfView: PDFView, willEndDisplayingOverlayView
        overlayView: UIView, for page: PDFPage) {
        let overlayView = overlayView as! PKCanvasView
        let page = page as! MyPDFPage
        page.drawing = overlayView.drawing
        pageToViewMapping.removeValue(forKey: page)
    }
}
```

demo

## Saving your work
* Use PDF Annotations as the model
* PDF annotations can have an "apperance stream"
	* nearly anything you can draw with Quartz2D
	* everything else can be rendered into an image, e.g. metal
* Since they are dictionaries, we can store them as custom data, too

```swift
// Implement a subclass with a drawing override

class MyPDFAnnotation: PDFAnnotation {
    
    override func draw(with box: PDFDisplayBox, in context: CGContext) {
        UIGraphicsPushContext(context)
        context.saveGState()
        
        let page = self.page as! MyPDFPage
        if let drawing = page.drawing {
            let image = drawing.image(from: drawing.bounds, scale: 1)
            image.draw(in: drawing.bounds)
        }
        
        context.restoreGState()
        UIGraphicsPopContext()
    }
}
```

```swift
override func contents(forType typeName: String) throws -> Any {
        
    if let pdfDocument = pdfDocument {
      
        // Go through all pages in the document
        for i in 0...pdfDocument.pageCount-1 {
            if let page = pdfDocument.page(at: i) {                       
                if let drawing = (page as! MyPDFPage).drawing {
                        
                    // Create an annotation of our custom subclass
                    let newAnnotation = MyPDFAnnotation(bounds: drawing.bounds,
                                                        forType: .stamp, withProperties: nil)
                        
                    // Add our custom data
                    let codedData = try! NSKeyedArchiver.archivedData(withRootObject: drawing,
                                                                      requiringSecureCoding: true)
                    newAnnotation.setValue(codedData,
                                           forAnnotationKey: PDFAnnotationKey(rawValue: "drawingData"))
                        
                    // Add our annotation to the page
                    page.addAnnotation(newAnnotation)
                }
            }
        }

        // -- Option #1: Save the document to a data representation
        if let resultData = pdfDocument.dataRepresentation() {
            return resultData
        }
      
        // -- Option #2: Save the document to a data representation and "burn in" annotations
        let options = [PDFDocumentWriteOption.burnInAnnotationsOption: true]
        if let resultData = pdfDocument.dataRepresentation(options: options) {
            return resultData
        }
    }
                
    // Fall through to returning empty data
    return Data()
}
```

Toa ddress filesize, we have two new options
`.saveAllImagesAsJPEG` => just what it says.
`optimizeImagesForScreen` => downsample to a maximum of high DPI screen resolution.  May be used together.
`createLinearlizedPDF` => special PDF that's optimized for the internet.  PDF Is read from end, so the entireity needs to be downloaded first.  Linearized has the first page at the beginning of the file.

# Wrap up
* PDFKIt is powerful, yet easy to use
* Now has overlay view support


* https://developer.apple.com/forums/tags/wwdc2022-10089
* https://developer.apple.com/forums/create/question?&tag1=188&tag2=415030
* https://developer.apple.com/sample-code/wwdc/2017/PDFAnnotationWidgetsAdvanced.zip
