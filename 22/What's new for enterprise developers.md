Discover how you can build compelling apps for your business on iOS, iPadOS, macOS, and watchOS. We'll take you through a curated overview of the latest updates to Apple platforms and explore relevant features that you can use to create engaging enterprise apps to transform workflows, inform business decisions, and boost employee productivity.

A lot of announcements this year.

# Siri
This year we announced App Intents, an all-new framework making it faster and easier to build great things for you app.
No intent definition file or generated code.

Upgrade with 'convert to app intent' button
Used to create app shortcuts.

## Shortcuts:
Installed alongside the app
Invoke shortcuts with zero setup
Parameterized trigger phrases
'add to siri' button no longer needed
Users will still need to be made ware of phrases.  Two ways:
* siri tip
* shortcuts link button

limited to 10 predefined app shortcuts.

[[Dive into App Intents]]
[[Implement App Shortcuts with App Intents]]
# Widgets

* Use SiriKit intents for widgets
* Lock screen widgets enable new experiences
* Complications now use a widget extension
* Code sharing between iOS and watchOS

New in iOS 16, widgetkit can be used to build widgets for the new lock screen and complications on apple watch.

iOS/watchOS that replace previous watchkit complications.

Few common widget types to choose from.

* rectangular
* accessoryCircular
* accessoryInline
* accessoryCorner (watch only)

New rendering modes to support new system styles in iOS 16, to ensure your widget feels at home.

## Live activities

* real-time data
* Display on lock screen or dynamic island
* WidgetKit and SwiftUI for the interface
* All-new ActivityKit framework (16.1)
* Remote notifications for data updates

Persistently displays without unlocking the phone.  
on pro/promax, see on the lock screen and also in dynamic island.

interact with realtime data across the system at a glance.  See relevant data for jobsite, etc.

[[Complications and Widgets Reloaded]]
[[Go further with Complications in WidgetKit]]

Lock screen widgets
clockkit complications are deprecated
code sharing between iOS and watchOS
live activities

# Augmented reality

ARKit 6.  4K video mode.
Camera enhancwements
plane anchors
Motion capture 
location anchors


roomplan
swift api powered by arkit
real-time scanning with lidar
create 3d floor plans
parametric representation

[[Discover ARKit 6]]
[[Create parametric 3D room scans with RoomPlan]]

# Vision

Enable your users to quickly scan for relevant info with juts a camera.

New revisions
* text recognition
* barcode scanning
* optical flow

Text recognition for korean and japanese.
automatic language recognition
but configure it if you know for better perf

## Live text API
iOS 16 and macOS 13
This year we announced new swiftapi to bring live text to your own apps.
Static images and paused video frames
Text interaction
Data detection
Translation
QR codes

DataScannerViewController in visionkit
Reduced implementation time
Text and machine-readable codes
Live camera preview
Guidance and item highlighting
Tap to focus and pinch to zoom

Text scanning, street address, duration, etc.

[[What's new in Vision]]
[[Capture machine-readable codes and text with VisionKit]]

Use the latest revision!

# Maps
* new map and 3d city experience
* LookAround API
* Selectable map features
* Overlay enhancements
* Map Configuration API

New property for a preferred configuration
3 configuration types:
* imagery
* hybrid
* standard

Replaces map type and associated properties (now deprecated!)

## server APIs
* geocoding
* search
* ETA

[[What's new in MapKit]]
[[Meet Apple Maps Server APIs]]

# Weather

Powered by Apple Weather Service
native framework or REST API
High-resolution weather models
Machine learning and prediction algorithms
hyperlocal weather forecasts globally
Respects privacy

Current conditions
10-day hourly forecasts
Minute forecast of precipitation for next hour
severe weather alerts
historical weather 

[[Meet WeatherKit]]

# Push to talk

Supports 'walkie-talkie' style communication
provides consistent and accessible UI
Receive and play audio from the background
Reliable and power efficient
Great fit for first responders, retail, and warehouse environments

[[Enhance voice communication with Push to Talk]]

# CarPlay

Maps, turn by turn instructions.  Instrument cluster located in front of driver.  Field service, sales, delivery, and transit

CarPlay entitlements usd to be limited to certain app types.
This year we're adding fueling and driving task apps.

Control car accessories, etc.  Beginning and end of a drive, such as mileage.

CarPlay simulator, a standalone mac app.

[[Get more mileage out of your app with CarPlay]]

# UI frameworks
## Swift charts
* flexible framework to create charts
* Uses same syntax as swiftUI
* Supports localization
* Built-in accessibility support
* Multi-platform

## Desktop-class iPad apps
* find and replace UI
* Enhanced search functionality
* Desktop-class editing
* New navigation bar styles.  Denser, more customizable layout
	* Navigator -> push top navigation model.  Hierarchical data, such as settings
	* Browser -> safari, files, viewing and navigating back and forth between multiple documents
	* Editors -> focused viewing or editing of individual documents

Navigation split view.
* perfect for multi-column apps
* adapts to a single-column stack on iPhone.
* Two-column layout.  Even a 3 column layout like notes.

Allows for easy implementation of deep linking and programmatic navigation.
* navigation links -> presents other views in swiftUI.
* Previously, you navigated based on
	* destination view
* now also based on presented data value.
* Navigate programmatically.

## Device name
UIDevice.name now returns the device model.
Some apps can still use the user-assigned name.
This requires an entitlement.  When requesting entitlement, let us know if it's a shared device.
Review docs for eligibility criteria.

[[Hello Swift Charts]]
[[The SwiftUI cookbook for navigation]]

# Design

**Great design matters.**

HIG was fully redesigned/refreshed.  Merged into a unified document.  Simpler to explore common design approaches while still preserving relevant details about each platform.

Later this year, we're adding changelogs for the entire set of guidelines.


700 new symbols.  More than 4k symbols to choose from, and all are available directly in Xcode, or from SFSymbols app.

Rendering modes give you control over how color is applied.
* monochrome
* hierarchical
* palette
* multicolor

automatic rendering is the new default.  
To visualize how they look in different rendering modes, SFSymbols app gained a new preview area, located in the right sidebar.  Adds support for variables.  See variable collection.  Vary the layers from a value from 0-1.

Unified layer annotation.  This allows you to have a shared layer structure across rendering modes, making annotation faster and easier.

## expanded SF font family
3 new width styles
* condensed
* compressed
* expanded

SFArabic, SFArabic Rounded.

we're now only requiring a single large-format image for iOS.  We scale it for display.  Can add custom images for smaller sizes.

[[What's new in SF Symbols 4]]
[[Meet the expanded San Francisco font family]]

# Wrap up
* watch sessions in the developer app
* Use the latest SDKs
* Provide feedback
* 