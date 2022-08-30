#coremedia

Modern camera driver architecture.  This is the replacement for DAL plugins.

Core Media IO's device abstraction layer.  Since macos 10.7

The power to extend macOS as a rich media platform.  Part of what makes the mac the mac.

# Dal plugins have some problems
* pose serious security risks
* Don't work with apple apps for this reason
* Don't work with many 3rd party camera apps
	* unless they disable library validation ro the user turns off SIP
* Difficult to develop
* Sparsely documented

It's time for an upgrade.  macOS 12.3 introduces a thoroughly modern replacement called camera extensions.  An architecture that places user security first.


# Technology overview

## Camera extensions
e.g. coremedia IO extensions are a new way to package/deliver camera drivers.
* secure
* Fast
* Modern
* Simple
* Deployable
* Compatible.  Even in apple apps.

## Pure software camera
color bars, test pattern, 1.
programmatically-generated images
prerendered content.  

## Hardware camera
Hotplugging and unplugging.
* driver kit extension (DEXT)
* IOVideoFamily kext (legacy)

Class-compliant extension for USB video class (UVC) cameras.  If however you need to support USB that uses nonstandard protocol, has additional features out of spec, createa a camera extension that overrides our UVC extension, claiming a particular product or vendor ID.

Please refer to the article "overriding the default USB video class extension".

## Creative camera
Maybe you access another video stream, apply ane ffect, and send them along to clients as a new camera stream.

Or, a creative camera that multiplexes several cameras.

Might use a configuration app to control compositing or filter settings, etc.

## Core Media IO
#coremediaio => low-elvel frmework for publshiing and discover
* dal plugin api and c++ sdk
* Core media IO extensions API
* App-side C API

## extensions
Built on top of the system exensions framework #systemextensions
First appeared in macOS 10.15.
App houses extension
Installs extension for all users
Deleting the app uninstalls the extension
Deployable in the app store!

See documentation for system extensions framework.

Check out [[System Extensions and DriverKit - 19]]

# building a camera extension
demo
OSSystemExtensionREquestDelegate implementation.
System Extension key in entitlements.  Required if our app installs system extensions.

File=>New=>Target=>macOS=>Camera Extension.

Four new files
* info.plist => identifies as CMIOExtension, mach service name.  CoreMedia IO's register assistant will not launch your extension unless it's present.  Let's also give i  a usage description.
* Entitlements file => app sandbox is true.  Need to ensure that my extension's app group is prefixed by the mach service name in order to pass validation.  So I pass that over from the app entitlements to extension entitlements.

System extensions can only be installed by apps residing in `/Applications`.

`systemextensionsctl list`.

removing the app will remove the extension.  

Let's turn that software camera into a creative camera.

## CIFilterCam
Creative camera support (using a hardware camera)
* requires macOS 13
* `com.apple.security.device.camera` in Entitlements
* NSCameraUsageDescription in Info.plist.
# The APIs
1.  Daemon process, e.g. coremedia io extensiosn
2. within an app process
	3. CoreMedia IO extension code
	4. core media IO extension -> DAL plugin shim
	5. Core Media IO App APIs (public)
	6. AVFoundation capture APIs #avfoundation 

Legacy plugins => DAL plugins may or may not be trusted.  These run directly in the app process, making it vulnerable to malware.  Camera extensions remove this attack vector.

* app sandbox required
* Launched under the `_cmiodalassistants` role user
* Subject to a custom sandbox profile
	* Communicates over USB, bluetooth, wifi (as a client, not a server that opens ports)
	* FireWire
	* Read and write from cache, container, tmp
	* Sandbox profile disallows
		* forking/exec/posix spawn
		* window server access
		* Connection to user account
		* Registering mach services in global namespace
		* If as you're developing, you find the sandbox too restrictive for a legitimate capture case, please provide a feedback in feedback assistant

We welcome your feedback

Between your daemon and the app is another process.  Proxy.  `registerassistantservice`.  Transparency, consent, control policy.  When an app uses a camera for the first time, it asks the useer.  needs to be granted for all cameras.  Handles this consent on your behalf.

Handles attribution.  Lets the system know that a parituclar camera is in use.  Power consumed by your daemon can be attributed to the app using it.


* CMIOExtensionProvider
* CMIOExtensionDevice
* CMIOExtensionStream
all 3 have CMIOExtensionProperties

Create each class by providing a ProviderSource, DeviceSource, StreamSource respectiely.

Provider:
* your daemon's service
* add remove devices
* aware of all clients
* consults your provider source for properties

provider source:
properties
* providerManufactureer
* providerName

device:
* manages streams (add/remove)
* be aware avfoundation ignores all but first input stream
* created with
	* device source
	* localized name
	* deviceID (UUID)
	* legacyID (optional)

| CMIOExtensionDevice | AVCaptureDevice     |
| ------------------- | ------------------- |
| `.localizedName`    | `.localizedName`    |
| `.deviceID`         | `.uniqueIdentifier`, unless |
| `.legacyDeviceID`   | `.uniqueIdentifier`                    |

Only provide the legacy ID if you're providing compatibility with a DAL plugin.

| CMIOExtensionDevice              | AVCaptureDevice  |
| -------------------------------- | ---------------- |
| `.deviceModel`                   | `.model`         |
| `.deviceIsSuspended`             | `.isSuspended`   |
| `.deviceTransportType`           | `.transportType` |
| `.deviceLinkedCoreAudioDeviceID` | `.linkedDevices`                 |

CMIOExtensionStream
* publishes formats
* Configures the active format
* Usesa  standard or custom clock
* Sends sample buffers

| CMIOExtensionStreamSource  | AVCaptureDevice                |
| -------------------------- | ------------------------------ |
| `.formats`                 | `.formats`                     |
| `.streamActiveFormatIndex` | `.activeFormat`                |
| `.streamFrameDuration`     | `.activeVideoMinFrameDuration` |
| `.streamMaxFrameDuration`  | `.activeVideoMaxFrameDuration`                               |

## What's missing
NO replacement for DAL control API
* boolean, selector, feature controls
* Powerful, but inconsistently implemented

Getting custom properties to the app layer. Use `CMIOObjectPropertyAddress`.
selector => 4-character code
scope=> global, input, output
element => any number you want.  Main element is always 0.

Bridge your properties by coding property address elements into a custom property name.`4cc_selector_scope_element`.  Communicate any string or etc.

AVFoundation does not support custom properties, your conifugration app must stick with C API.
# Output devices
Apps can provide video *to* them.  This is the **O** in core media I**O**

* print to tape
* real-time preview monitoring
* No AVFoundation support (Core Media IO only)

`CMIOExtensionStream` has direction
* source provides frames
* sink streams consume frames
Clients tell your stream to `consumeSampleBuffer`
You tell clients you've consumed it with `notifyScheduledOutputChanged`

* `sinkBufferQueueSize`, `sinkBuffersRequiredForStartup`, etc.

# DAL plug-in deprecation plan
> Camera extensions are the path forward

DAL plugins are already deprecated (macOS 12.3).  So, you get a compilation warning when building.

As long as legacy DAL plugins are allowed to load, camera apps will still be at risk.  To fully address security vulnerabilities, **macOS 13 will be the last to support DAL plugins.**

## Call to action
* port existing DAL plugins to camera extensions
* provide feedback with feedback assistant


* https://developer.apple.com/forums/tags/wwdc2022-10022
* https://developer.apple.com/forums/create/question?&tag1=281&tag2=77&tag3=384030
* https://developer.apple.com/documentation/coremediaio/overriding_the_default_usb_video_class_extension
* https://developer.apple.com/documentation/coremediaio/creating_a_camera_extension_with_core_media_i_o
* https://developer.apple.com/documentation/coremediaio
* https://developer.apple.com/system-extensions/
* https://developer.apple.com/documentation/avfoundation/capture_setup
