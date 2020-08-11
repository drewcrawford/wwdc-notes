#applesilicon 

[[Introducing iPad apps for mac - 19]] - more on catalyst

# Automatic behavior
Get a tremendous amount of behavior automatically.
However, they don't get access to everything macs have to offer.

# Compatibility
Your app can't be dependent on an unavailable symbol/framework, missing functionality, no dependence on misisn ghardware capabilities

# Available
Compatible apps are automatically available
Manage availability in App Store Connect.  Might exclude if you already have a mac app.

# Environment differences
* Hardware
* UI
* system software

## Hardware differences
* Mouse and touch events.  Custom touch handling may not be compatible.
* Supporting keyboard entry will help.
* Environment sensors.  GPS, compass, accelerometer, etc.
* Cameras.  Macs have more configurations.  Consider `AVCaptureDeviceDiscoverySession`.

## UI differences
* Automatic behavior.  However, open/save is in a separate window, alerts may be in a different location, etc.
* Window resizing.  (iOS multitasking)

## software
Filesystem.  Users may move the app.  
Device properties.  Better to ensure your device robustly handles unexpected values.
Screen size -> will not match an iOS device.

You can debug with the "Designed for iPad" target.
Can run XCTest unit tests natively on an apple silicon mac.

# Distributing
* Familiar workflows
* Mac is just another device
* Available on the mac app store

For new submissions, the standard export workflows work.

## app store features
IAP & subscriptions
all apps tore features

## thinning
App content selected for best fit for mac
Inapplicable resources left out for streamlined install
Just like another (very capable) iPad!

Xcode has a new Mac virtual thinning target
Optimized for all macs
Single app variant

# Developer or ad-hoc distribution
Just like iPhone & iPad

# Key considerations
TestFlight not available for Mac.  Need to use ad-hoc or development distribution
There are limits to iPhone/iPad app runtime compatibility on mac
Test app to evaluate user experience.

# iPad and iPhone apps on Mac
Effortless distribution
Tooling
Distribution control
