# Overview
* reliable and secure
* easy to develop
* Distributed on the app store

We now supprot entworking,  block storage, serial, audio, and SCSI controller and peripheral drivers, in additional to USB, PCI, and HID.  

[[Creating audio drivers with DriverKit]]
[[Modernize PCI and SCSI drivers with driverkit]]


# AudiodriverKit updates
## Real-time operations
* Register a realtime callback with ADK
* get a callback every time an IO op happens
* Use for signal processing.

## Entitlements
`com.apple.developer.driverkit.allow-any-userclient-access`
we itnroduced a new entitlement
`com.apple.developer.driverkit.family.audio`.  Don't use the other one.
But you can keep it if you want any app to communicate with your driver.

```cpp
// Declare a IOOperationHandler block to set on the IOUserAudioDevice.
// The block will be called from a real time context when a i/o operation
// occurs on the IOUserAudioStream buffers for the device.
io_operation = ^kern_return_t(IOUserAudioObjectID in_device,
                              IOUserAudioIOOperation in_io_operation,
                              uint32_t in_io_buffer_frame_size,
                              uint64_t in_sample_time,
                              uint64_t in_host_time)
{
    // Add custom code to make modifications to the buffers as necessary
    if (in_io_operation == IOUserAudioIOOperationWriteEnd) {
        ...
    } else if (in_io_operation == IOUserAudioIOOperationBeginRead) {
        ...
    }
    return kIOReturnSuccess;
};
this->SetIOOperationHandler(io_operation);
```

this is public **for development**
all driverkit environments are available for development

to distribute, see https://developer.apple.com/contact/request/system-extension


# DriverKit on iPad

Excited to announce that we're coming to iPad!  Possible to extend the system in a safe and secure way.

**No changes required for existing drivers**.

* USBDriverKit
* PCIDriverKit
* AudioDriverKit

> This is made possible with the power of the M1 chip.  All iPads with M1 will support DriverKit

* build the same driver everywhere
* Use multiplatform apps
* Automatically sign and provision
* Distribute on the app store
[[Use Xcode to build a multiplatform app]]

## Demo
* automatic discovery on iPadOS
* SystemExtensionf ramework on macOS
Since drivers are low-level software, they need to be approved by the user before they can run.  Users need to go through system preferences on macOS.
on iPadOS, driver approvals are in the settings app.

General settings will have alist of all available drivers.
If you have a settings bundle, there's a drivers link inside your settings.
Prompt the user to enable the driver by visiting settings.

Add iPad support.  

Use platform support to conditionally compile per-file in build settings.

`UIApplication.openSettingsURLString`.

Keep in mind that drivers launch on demand.  Driver only starts running when hardware device is plugged in.  

Wireless debugger, etc.

## User clients
Allow your appt o communicate with your driver
Use IOKit.framework to open user clients
See sample code.

We know apps acn communicate with drivers, but keep security in mind.  We don't want to allow every app to communicate with a driver.

app needs driverkit userclient-access entiltment.  Value is array of allowed driver bundle identifiers.

On iPadOS, we added a enw entitlement called "Communicates with drivers". It replaces macos entitlement.

`com.apple.developer.driverkit.communicates-with-drivers`.

Allow apps from other developers to interact?  Each app needs the communicates with drivers entitlement.  Driver needs Allow Third Party User Clients entitlement.

Without this entitlement, only apps from the same team can communicate with the driver.  

## Entitlements
* Public for development
* Request for distribution

see link

## Updates
For apps containing drivers, updates are different.

Driver approval state is maintained through updates.  However, since ahrdware is still plugged in, prior driver may still be running.  Since old driver continues running, v2 app may have to communicate with v1 driver.  

When device is unplugged, v1 will get killed.  Can now be updated.  
* app is updated anytime
* driver is updated after device is unplugged
* new app may communicate with old driver

# Distribution
* include your driver inside an app on the app store
* drivers are only loaded on compatible devices
	* UIREquriedDeviceCapabilities for driverkit.  Prevents users from installing app on other device.
* We suggest submitting a video to app review

* https://developer.apple.com/documentation/driverkit/communicating_between_a_driverkit_extension_and_a_client_app
* https://developer.apple.com/documentation/kernel/implementing_drivers_system_extensions_and_kexts
* https://developer.apple.com/system-extensions/

