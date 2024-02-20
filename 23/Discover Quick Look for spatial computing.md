Learn how to use Quick Look on visionOS to add powerful previews for 3D content, spatial images and videos, and much more. We'll show you the different ways that the system presents these experiences, demonstrate how someone can drag and drop Quick Look content from an app or website to create a separate window with that content, and explore how you can present Quick Look directly within an app.

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