Learn how to use Quick Look on visionOS to add powerful previews for 3D content, spatial images and videos, and much more. We'll show you the different ways that the system presents these experiences, demonstrate how someone can drag and drop Quick Look content from an app or website to create a separate window with that content, and explore how you can present Quick Look directly within an app.

All the capabilities of QL on xrOS.  Unlock new experiences for your app/website.  Before we begin, let's review quicklook.

A Framework on macOS, iOS, and now xrOS
Easily preview files within your app
Powerful editing features for images, PDFs, and media
Secure and private

System viewer for file-backed content.  xrOS can leverage QL to easily gain editing, etc.

Just drop file wherever.  dropping presents a 3d ql preview where I dropped it.  Immediately start to interact with it, etc.

# Windowed QL
View your content alongside your app.  Since windows QL previews are hosted in a separate app, you can close your application while still ahving previews persist.  Much richer viewing experience for 3d models.

Scale/place 3d content independent of your application.  

Provide a URL to a drag provider.  System copies the URL, presents copy in QL window.  
System will download URLs if needed.  Because a copy of the file is presented, markups on image will NOT be written back to the URL.

### 5:15 - drag support for quick look from apps

```swift
import Foundation
import SwiftUI
import UniformTypeIdentifiers

struct FileList: View {
    
    @State var files: [File]
    @State var previewedURL: URL? = nil
    @State var selectedFile: File? {
        didSet {
            self.previewedURL = selectedFile?.url
        }
    }
    
    var body: some View {
        List(files, selection: $selectedFile) { file in
            Button(file.name) {
                selectedFile = file
            }
            .onDrag {
                return NSItemProvider(contentsOf: file.url) ?? NSItemProvider()
            }
        }
    }
}
```

Linked 3d content will present in a QL window

Websites already linking AR content will get this behavior for free

Respects cutomization options

[[Advances in AR Quick Look - 19]]

```html
<a rel="ar" href=""> <img src="/"> </a>`
```


# In-app QL
Use quickLookPreview to present a fullscreen preview.
QLPreviewController for more customization options
Existing in-app QuickLook uses will 'just work'

supported file types:
images, videos, audio, pdf, html, rtf, public.text, iWork/office, zip, usdz



### 8:45 - swiftUI quick look preview function

```swift
import Foundation import SwiftUI
struct FileList: View {
  
@State var files: [File]
@State var previewedURL: URL? = nil
@State var selectedFile: File? {
	didSet {
		self.previewedURL = selectedFile?.url
		}
  }
  
var body: some View {
	List(files, selection: $selectedFile) { file in
			Button(file.name) {
				selectedFile = file
			}
		}
		.quickLookPreview($previewedURL, in: files.map { $0.url })
  	}
}
```

# Takeaways
* windows QL to preview content alongside your app
* immersive 3d content viewing
* shareplay
* In-app QL for linear workflows
* Existing uses of QL will just work