Discover the latest updates to the Virtualization framework. We'll show you how to configure a virtual machine (VM) to automatically resize its display, take you through saving and restoring a running VM, and explore storage and performance options for Virtualization apps running on the desktop or in the data center. To learn more about the Virtualization framework, check out “Create macOS or Linux virtual machines” from WWDC22.

Configure, manage, run VMs.  Virtual macs and linux vms are more powerful than ever.  New configuration options are highly customizable.

many features make this possible.

# User experience
* resize a display
* save a VM
## resize a display

screen size is a big part of workign on a virtual mac.  With resizable display, VM dynamically adjusts screen resolution to fill the window.  Making most efficient use of your space.

VZVirtualmachineView.  Choose `automaticallyReconfiguresDisplay = true`.

## save/restore a VM.

powering off the VM.  Starting means doing a cold boot.  But when working on a mac, most applications save your progress as you work.  Same should hold for VMs.

```swift
try await virtualmachine.pause()
try await virtualMachine.savemachineSTateTo(url:)
try await virtualMachine.restoreMachineStateFrom(url:)
```

same state as before!

### 1:58 - Set display as resizable
```swift
// virtualMachine is a VZVirtualMachine.
let virtualMachineView = VZVirtualMachineView()
virtualMachineView.virtualMachine = virtualMachine

virtualMachineView.automaticallyReconfiguresDisplay = true
```

### 4:20 - Save a virtual machine
```swift
// virtualMachine is a running VZVirtualMachine.
try await virtualMachine.pause()

let saveFileURL = URL(filePath: "SaveFile.vzvmsave", directoryHint: .notDirectory)
try await virtualMachine.saveMachineStateTo(url: saveFileURL)
```

### 4:58 - Restore a virtual machine
```swift
let configuration = VZVirtualMachineConfiguration()
// Customize configuration.

let virtualMachine = VZVirtualMachine(configuration: configuration)

let saveFileURL = URL(filePath: "SaveFile.vzvmsave", directoryHint: .notDirectory)
try await virtualMachine.restoreMachineStateFrom(url: saveFileURL)

try await virtualMachine.resume()
```



when restoring from file
* hardware encrypted.  No other mac or user account can read this.
* versioned.  New capabilities can be added in the future.  If the file format changes, and we can't restore, we return some error codes.  We recommend discarding the file and rebooting if this happens.



# Configuration options
* network block device
* NVMe controller devices.
* Mac keyboard

## network block device
in VM framework,s torage is typically served locally.  Reading/writing disk image on the same mac, etc.

In sonoma, virtualizationf ramework can serve storage remotely from a server.  Protocol that allows this is Network Block Device (NBD).

virtualization framework implements the client side of nbd protocol.  When a vm access the disk, request is forwarded to NBD server.

NBD server responds with data.  Extremely flexible.

1.  Storage can now reside anywhere.  On same mac, or on a remote server over the network
2. Since storage is managed by your own server, implement whatever custom IO you need.  Custom image formats, physical drives, etc.

VZVirtioBlockDeviceConfiguration
VZUsbmassStorageDeviceConfiguration

storage attachment -> VZDiskImageStorageDeviceAttachment
VZNetworkBlockDeviceStorageAttachment

### 9:28 - Configure a Virtio block device with the NBD attachment
```swift
let url = URL(string: "nbd://localhost:10809/myDisk")!
let attachment = try VZNetworkBlockDeviceStorageDeviceAttachment(url: url)

let blockDevice = VZVirtioBlockDeviceConfiguration(attachment: attachment)
```

### 10:02 - Respond to events with a delegate with the NBD attachment
```swift
let url = URL(string: "nbd://localhost:10809/myDisk")!
let attachment = try VZNetworkBlockDeviceStorageDeviceAttachment(url: url)

// NetworkBlockDeviceAttachmentDelegate implements the delegate protocol.
let delegate = NetworkBlockDeviceAttachmentDelegate()
attachment.delegate = delegate

let blockDevice = VZVirtioBlockDeviceConfiguration(attachment: attachment)
```

## NVMe
Standardized technology enabled in many linux kernels.  Applications are more specific.

Why to use?

For vast majority of usecases, virtio is the way to go.  Some linux VMs don't have virtio drivers.  These may be kernels not built to run in virtual environments.  These kernels may have NVMe drivers.  In macOS sonoma, NVMe devices are emulated by virtualization framework.  Allowing more operationg systems to run in VMs.

To configure, u se device type.

* linux virtual machines (only)
* also supports NBD

## mac keyboard
Use globe key for language switching, etc.

# Rosetta 2

In a linux VM is the same tech used on macOS.  In macOS sonoma, performance improvements mean rosetta 2 runs even faster in a virtual linux environment.

How is this possible?  AOT compilation vs JIT.  

Let's say we have multiple apps that link the same DLLS.  Even though we've already translated it in one executable contexxt, have to do it again.  Rosetta 2 solves this with caching.  We share results with apps that need it, so we don't need to retranslate.

Now we bring this optimization to linux VMs with a new runtime daemon.

* configure communication channel
* launch daemon in virtual machine

dynamic linker requests are forwarded to demon.  Rosetta fetches pretranslated binaries from demon.  Biggest impact on exec-heavy tasks like compilation or package installation

* resizable display
* mac keyboard
* save a vm
* network block device







# Resources
* https://developer.apple.com/documentation/virtualization/running_macos_in_a_virtual_machine_on_apple_silicon
* https://developer.apple.com/documentation/virtualization
