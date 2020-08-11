# Recap
[[System Extensions and DriverKit - 19]]

* more reliable
* more secure
* easier to develop

Run in userspace, not kernel space.  

## advantages over kexts
* user space
* restricted access
* separate from the rest of the system and hardware
* no need to restart with crashes
* build, test, debug on one machine

kexts:
* kext bugs compromise the entire kernel
* access to entire system
* kernel panics
* two-machine debugging
	* special debug cables or LAN setup
* slow dev process

## installation
No installer or package necessary
extensions stay in app bundle
Do not manually copy driver extensions to system folder
Use `SystemExtensions` framework to install
Most apps should submit an `activationRequest` at app launch.

## prototyping driver extensions
disable SIP
entitlements

## new
PCI (10.15.4)
scsi

Deprecating kernel extensions
macOS Catalina was the last release to fully support kexts without compromises
Using a kernel extension to perform the same function is deprecated
In macOS Big Sur, kexts that were deprecated in macOS catalina will not load by default

More Driverkit device families will be added, more kernel extensions will be deprecated.
PCI and SCSI controller kernel drivers are deprecated
They will not load in a future macOS release


# PCIDriverKit
## PCIe
* capability expansion
* high bandwidth
* high performance

PCIE cards can be added through
* thunderbolt devices
* thunderbolt chasses
* PCIe slots

All driver extensions: `com.apple.developer.driverkit`
PCI driver transport entitlement `com.apple.developer.driverkit.transport.pci`
`IPOCIFamily` matching criteria

`com.apple.developer.driverkit.family.networking` -> for a NIC driver

`IOPCIDevice`.  Don't subclass.  This has a userspace instance and a kernel-space instance.  Apparently the userspace instance is a proxy.

## session management
PCI driver extensions must open their PCI provider prior to memory access
One client per IOPCIDevice
State is *not* maintained after closing the session.

## memory access
All memory access is done through IOPCIDevice
Device memory mapping is done by PCIDriverKit
Enhances debuggability

## memory indexes
for 64-bit bars, a `memoryIndex` points to 2 bars.

## configuration access

## examples
Not copying these
* read/write configuration, start/stop
* interrupt handler
* Direct memory access


# SCSIControllerDriverKit
Available today in macOS Big Sur
Replacement for `IOSCSIParallelInterfaceController`.
Great performance
Can be used to build device drivers for:
* fibre channel controllers
* RAID controllers
* Serial attached SCSI

New interface in `IOUserSCSIParallelInterfaceController.iig`
Similar API structure to kenrle class with "User" prefix
ex, `UserProcessParallelTask()` 
All pure virtual functions are to be overridden by the dext
These functions are calls made by the framework into the dext

## Entitlements
Entitlements required:
DriverKit entitlement
Transport entitlement
Family entitlement `com.apple.developer.driverkit.family.scsicontroller`

## Creating a SCSI text
### start/initialization
kernel controls this
Kernel gathers controller specific information
* maximum task count supported
* Highest supported device ID

Kernel Creates process hosting the dext
Calls `Start()`
Calls various other functions `UserInitializeController`, `UserStartController`, etc.

How to implement `UserInitializeController`
1.  Create session
2.  Set up some auxillary queues
3.  register for interrupts

### dispatch queueing model
We suggest having 3 dispatch queues
1.  Default queue (created for you).  This runs calls that originate from the kernel.
2.  Interrupt queue.  Helps to have a separate queue, so that interrupts and I/Os won't be competing with each other to run.  Can also be used to service dext I/O timeout handlers.
3.  auxilliary queue.  Create SCSI targets.  `UserCreateTargetForID()`.  Dext needs to create the scsi target from a separate queue.

## I/O path
### kexts
Previously, kext overrides `ProcessParallelTask`.  Then calls `CompletePrallelTask` with results.
### dext
Kernel will now call `UserProcessParallelTask`.  Will copy the object to userspace.  Kernel now handles more stuff?  This avoids various XPC.
Needs to call `ParallelTaskCompletion` at some point.

## power management
Override `IOService::SetPowerState()`.  
Power capabilities for PCIe based device
* off
* on
* lowpower

Delayed acknowledgement for power state transition is possible.
## Termination
Override `Stop()` method.
Framework handles completion of outstanding I/Os

# Demo

# Wrap up
* Introduced PCIDriverKit and SCSIControllerDriverKit
* Wrote a SCSI driver over PCI
* System extensions are the replacement for kexts
	* SCSI controller and PCI kernel extensions that can perform the same functions are deprecated

[[System Extensions and DriverKit - 19]]
