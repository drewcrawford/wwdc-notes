#classkit

# ClassKit Overview
Surface educational content as activities
Report progress
Protects the privacy of the students
Also available on #macOS

[[What's new in classkit - 19]]
[[Introducing ClassKit - 18]]

## App activity
Assignable content
Defined by your app
Structured hierarchically.  e.g. chapter, section, etc.
`CLSContext`.

Recommended thumbnail image method for coregraphics

```swift
func thumbnailFromImage(atURL: URL) -> CGImage? {
	if let imageSource = CGImageSourceCreateWithURL(atURL as CFURL, nil) {
		let thumbnailOptions = [kCGImageSourceCreateThumbnailFromImageAlways as String: true, kCGImageSourceThumbnailMaxPixelSize as String:330]
		return CGImageSourceCreateThumbnailAtIndex(imageSource,0,thumbnailOptions as CFDictionary)
	}
	return nil
}
```

Audience - completion time, age range
Progress types - percentage, hints used, etc.

# Enhancements to ClassKit


# Best Practices
Create full-featured contexts.
Update previously saved contexts.  Delete prior contexts.
Update context provider app extension

# More enhancements
new CLSContextType `course` and `custom`.

Now has non-assignable activities.  E.g., for a container etc.

# ClassKit Catalog API
* Improve discoverability
* Support more activities
* Update content anytime - decouple management from application

Your app -> classkit -> devices

Translate your CLSContext hierarchy into json, and then upload.

We intend this for public content, private content should use the native API.

`api.classkit-catalog.apple.com/v1`

Contexts and thumbnails.

`environment=development` 

 ## Guidelines
 No more than 200 contexts per POST.  If you have more, split them up across multiple requests
 Make sure you do the root first
 Deletion cascades to children.
 
 ## Thumbnail guidelines
 * Use PNG or jpg
 * 330x330
 * Upload contexts before thumbnails

## Authentication
JWT
Apple developer profiles, provision a new key

Can change to developer environment in system settings.

Be aware that content pushed to development environment are available to all developers
 
# Best practices for Classkit Catalog API

* Keep content fresh
* Localize your data
* Provide rich contexts and associated thumbnails

