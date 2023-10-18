#swift  #cpp 
Learn how you can use Swift in your C++ and Objective-C++ projects to make your code safer, faster, and easier to develop. We'll show you how to use C++ and Swift APIs to incrementally incorporate Swift into your app.

# Basics of interoperability
* builds on objc interop
* supports incremental swift adoption

Adopt swift in large c++ codebases
use c++ libraries in swift
remove objc bridging layers

demo
## photo editing UI stack
* image processing framework - c++
* user interface code - objc++
* swiftUI photo picker

no bridging header required
enable c++ interoperability in build settings "C++ and Objective-C interoperability".  It's C/ObjC by default.  But I can change it to C++.

cmd-click on module name to see its contents.

Struct must be public to accesss from ObjC++.
import the header that swift generated.  Contains all public swift APIs.  Now that I've imported, I can start calling swift code from C++.

## 4:10 - Calling a C++ method from Swift
```swift
func loadImage(_ image: UIImage) {
    // Load an image into the shared C++ class.
    CxxImageEngine.shared.pointee.loadImage(image)
}
```

## 4:20 - Import a C++ framework
```swift
import CxxImageKit
```

## 4:45 - Import the Generated Header
```objc
#import "SampleApp-Swift.h"
```

## 4:57 - Calling a Swift method in C++
```objc
- (IBAction)openPhotoLibrary:(UIButton *)sender {
    // Construct SwiftUI view
    SampleApp::ImagePicker::init().present(self);
}
```

* incremental swift adoption
* completely automatic
* direct calls, no overhead

## apis available in swift
* collections: `std::vector`, `std::map`, user-defined
* templates: function templates and class template specializations
* Smart pointers: `std::shared_ptr`, user-defined

## swift apis available in c++
* members: methods, properties, and initializers
* generics: structs and enums
* standard library: `Array`, `String`, `Optional`
* Library evolution: resilient structures and classes

## xcode support
* code completion
* jump-to-definition
* global rename
* debugger support

adopt swift in even more places!

# Natural Swift APIs

Able to automatically import most C++ APIs, represent them as safe swift APIs.

* getters/setters -> methods
* value types -> structs
* reference types -> pointers
* operators -> operators
* unsafe methods -> not imported
* containers -> swift collections
* constructors -> initializers
* +more!

a function or method might use a c++ naming convention that doesn't feel natural in swift.  In these cases, you can use annotations to rename an imported function. 

## 8:22 - Using the SWIFT_COMPUTED_PROPERTY attribute
```cpp
int  getValue() const SWIFT_COMPUTED_PROPERTY;
void setValue(int newValue);
//imported as: var value: Int { get set }
```

## 8:42 - Using the SWIFT_SHARED_REFERENCE attribute
```cpp
struct SWIFT_SHARED_REFERENCE(retain, release) CxxReferenceType;
//imported as: class CxxReferenceType
```

## 8:52 - Using the SWIFT_RETURNS_INDEPENDENT_VALUE attribute
```cpp
SWIFT_RETURNS_INDEPENDENT_VALUE 
std::string_view netwrokName() const;
//imports as: func networkname() -> std.string_view
//something to do with making an unsafe api as safe?
```

demo.
returns a c++ vector.  What is a vector?

Swift types are value or reference.  In swift, structs represent value, classes are refeference.
By default, c++ are imported as value types.  

## imported value types
* short lifetimes, deep copies
* copy constructor and destructor manage lifetime
* unlike swift array that is copy-on-write, when swift copies a vector, it copies all the elements.


## 10:45 - Using a for-loop to iterate over a C++ std::vector in Swift
```swift
// Get every image out of the shared C++ class.
for image in CxxImageEngine.shared.pointee.getImages() {
    let uiImage = CxxImageEngine.shared.pointee.uiImageFrom(image)
    UIImageWriteToSavedPhotosAlbum(uiImage, nil, nil, nil)
}
```

## collections
* imports any type with `begin` and `end` methods as Swift Collection
* for loops, map, filter, etc.
* Swift Collections are safe!
* iterators are easy to introduce bugs!  avoid!
* swift APIS are safe even on c++!

We mark unsafe c++ apis as unavailable.

example type has reference semantics. c++, not as clear which types fall into which category.  c++ doesn't have a strong distinction between value and reference types.

* default -> value type import
* custom -> reference type import

foreign reference types: `SWIFT_SHARED_REFERENCE(retain,release)`
* enforces reference semantics
* removes `UnsafePointer` indirection.
* Manages lifetime with custom retain/release operations


## 13:54 - Import swift/bridging
```cpp
#import <swift/bridging>
```

## 14:01 - Applying the SWIFT_SHARED_REFERENCE attribute to CxxImageEngine
```cpp
struct SWIFT_SHARED_REFERENCE(IKRetain, IKRelease) CxxImageEngine {
    // ...
};
```

to make getters/setters feel more native, I can use the `SWIFT_COMPUTED_PROPERTY` attribute.  map the pair into a swift computed property.


## 14:53 - Applying the SWIFT_COMPUTED_PROPERTY attribute to getImages
```cpp
/// \returns all images that have been loaded into the engine. Includes any modifications that were
/// applied to the images.
SWIFT_COMPUTED_PROPERTY
inline std::vector<Image *_Nonnull> getImages() const;
```

## 15:06 - Updated for-loop using the "images" computed property
```swift
// Get every image out of the shared C++ class.
for image in CxxImageEngine.shared.pointee.images {
    let uiImage = CxxImageEngine.shared.pointee.uiImageFrom(image)
    UIImageWriteToSavedPhotosAlbum(uiImage, nil, nil, nil)
}
```

Many other annotations.  All you need to do to access them is import `swift/bridging`.
**available on all platforms**.  Including linux and windows.

## future directions
* evolve based on your feedback
* Opt-in to changes with interoperability versions
* send us feedback please
* designed by swift compiler workgroup.  
	* swift.org/documentation/cxx-interop
# wrap up
Use c++ apis automatically and safely
Incrementally adopt swift
Fine-tune imported APIs

[[23/What's new in Swift|What's new in Swift]]
[[Swift and Objective-C interoperability - 15]]


# Resources
* https://swift.org/documentation/cxx-interop
* https://developer.apple.com/documentation/Swift/MixingSwiftAndC++InAnXcodeProject
* https://developer.apple.com/documentation/Swift/UsingC++APIsInSwiftAndSwiftAPIsInC++
