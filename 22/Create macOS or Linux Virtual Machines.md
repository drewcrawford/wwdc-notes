Run macOS and linux in virtual machines on apple silicon.
# Overview
All the pieces.
Hardware => special apple silicon magic.
macOS kernel =>
Hypervisor framework => low level API to virtualize CPU/memory
Virtualization framework => high level API.  macOS (apple silicon) or linux (both host arch)

Configuration => like configuring a mac on the apple store!  memory, etc.
Virtual machine

```swift
var configuration = VZVirtualMachineConfiguration()
configuration.cpuCount = 4
configuration.memorySize = (4 * 1024 * 1024 * 1024) as UInt64
configuration.storageDevices = [newBlockDevice()]
configuration.pointingDevices = [newPointingDevice()]
```

```swift
let virtualMachine = VZVirtualMachine(configuration: configuration)
try await virtualMachine.start()
```
show the display
```swift
let virtualMachineView = VZVirtualMachineView()
virtualMachineView.virtualMachine = virtualMachine
```

Too many features to cover.  In this session we'll look at core capabilities, for everything else see the docs.

# macOS
Supports macOS on apple silicon.  We developed macOS and virtualization framework together.  Incredible efficiency.

## Configuring
first, we need the platform.  
* hardware model – which version we weant
* auxiliary storage – non-volatile memory
* machine identifier => unique number representing the machine
```swift
let platform = VZMacPlatformConfiguration()

let hardwareModel = VZMacHardwareModel(dataRepresentation: savedHardwareModel)
platform.hardwareModel = hardwareModel!

let auxiliaryStorage = VZMacAuxiliaryStorage(contentsOf: auxiliaryStorageURL)
platform.auxiliaryStorage = auxiliaryStorage

let machineIdentifier = VZMacMachineIdentifier(dataRepresentation: savedIdentifier)
platform.machineIdentifier = machineIdentifier!

configuration.platform = platform
```

boot loader.
```swift
configuration.bootLoader = VZMacOSBootLoader()
```

## Installing
1.  Restore image
2. Compatible configuration
3. Installer

```swift
let restoreImage = try await VZMacOSRestoreImage.latestSupported

try await download(restoreImage.url)
```

```swift
let requirements = restoreImage.mostFeaturefulSupportedConfiguration

guard let requirements = requirements else {
    // No compatible configuration.
    return
}

platform.hardwareModel = requirements.hardwareModel

configuration.cpuCount = requirements.minimumSupportedCPUCount
configuration.memorySize = requirements.minimumSupportedMemorySize
```

```swift
let virtualMachine = VZVirtualMachine(configuration: configuration)

let installer = VZMacOSInstaller(virtualMachine: virtualMachine,
                                 restoringFromImageAt: imageURL)
try await installer.install()
```



## Using your Mac
**RUN METAL IN THE VIRTUAL MACHINE** #metal 

```swift
let graphicsConfiguration = VZMacGraphicsDeviceConfiguration()
graphicsConfiguration.displays = [
    VZMacGraphicsDisplayConfiguration(widthInPixels: 1920,
                                      heightInPixels: 1200,
                                      pixelsPerInch: 80)
]

configuration.graphicsDevices = [graphicsConfiguration]
```

### Virtual trackpad
```swift
let trackpad = VZMacTrackpadConfiguration()
configuration.pointingDevices = [trackpad]
```

Both on host and virtual machine.


## Sharing files
ventura only?
```swift
let sharedDirectory = VZSharedDirectory(url: directoryURL, readOnly: false)
let share = VZSingleDirectoryShare(directory: sharedDirectory)

let tag = VZVirtioFileSystemDeviceConfiguration.macOSGuestAutomountTag
let sharingDevice = VZVirtioFileSystemDeviceConfiguration(tag: tag)
sharingDevice.share = share

configuration.directorySharingDevices = [sharingDevice]
```

special tag to automount => macOSGuestAutomountTag

## Demo!


# Linux
## Installing
```swift
let diskImageURL = URL(fileURLWithPath: "linux.iso")
let attachment = try! VZDiskImageStorageDeviceAttachment(url: diskImageURL, readOnly: true)
let usbDeviceConfiguration = VZUSBMassStorageDeviceConfiguration(attachment: attachment)

configuration.storageDevices = [usbDeviceConfiguration, createBlockDevice()]
```

EFI boot loader
* standard
* boot discovery
```swift
let efi = VZEFIBootLoader()
efi.variableStore = VZEFIVariableStore(creatingVariableStoreAt: storeURL,
                                       options: [])
configuration.bootLoader = efi
```

## Running
virtIO GPU 2D.  Provides surfaces to host macOS.

```swift
let virtioGPU = VZVirtioGraphicsDeviceConfiguration()
virtioGPU.scanouts = [
    VZVirtioGraphicsScanoutConfiguration(widthInPixels: 1280, heightInPixels: 720)
]

configuration.graphicsDevices = [virtioGPU]
```

## Demo

## Rosetta 2
x86-64 binaries
run your ARM linux distribution, and its x86-64 aps.
fast!

```swift
let rosettaDirectoryShare = try! VZLinuxRosettaDirectoryShare()
let directorySharingDevice = VZVirtioFileSystemDeviceConfiguration(tag: "RosettaShare")
directorySharingDevice.share = rosettaDirectoryShare

configuration.directorySharingDevices = [directorySharingDevice]
```

Tell the system to use rosetta to handle x86_64 binaries

```bash
mount -t virtiofs RosettaShare /mnt/Rosetta

sudo /usr/sbin/update-binfmts --install rosetta /mnt/Rosetta/rosetta \
  --magic "\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x3e\x00" \
  --mask "\xff\xff\xff\xff\xff\xfe\xfe\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff" \
  --credentials yes --preserve no --fix-binary yes
```

* Simple
* high performance
