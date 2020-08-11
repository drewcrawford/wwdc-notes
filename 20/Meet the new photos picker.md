# What is PHPicker?

PHPicker is the best way to access photo and video data from photos
All-new design
api
multi-selection
selectable types are filterable

private by default
No direct access
Won't prompt for library access
Provides user selected photos and videos only

PHPicker is out of process

# NEw API
`PHPickerConfiguration` -> `PHPickerViewController` -> `PHPickerViewControlelrDelegate`
`PHPickerFilter`

`PHPickerResult`
# Some features require photos kit
Editing
?

[[Handling Limited Photos Library]]

# best practices
* AssetsLibrary has been deprecated and is still deprecated
* Switch to PhotoKit
* UIImagePickerController is being deprecated
* Adopt PHPicker
* Multi-select support
* Consistent UI with photos app
* Allow the user to pick iamges or video without requiring photos library access
* 