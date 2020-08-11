#applesilicon 

# Apple Silicon and #macOS 
Intel-based macs
* multicore cpu
* maybe discrete gpu with separate memory
* t2 features
	* secure enclave
	* storage contorller,
	* motioncoprocessor
	* image signal processor

# apple SOC
* Unified memory architecture -> metal
	* Share textures efficiently without copying over pcie bus
* Video encoder and decoder -> AVFoundation, VideoToolbox.  Recommend `kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange`, e.g. Biplanar.
* Neural engine -> CoreML.  
	* Note that you don't want to use `.cpuOnly` or `.cpuAndGPU` since neither of them are the neural engine.
* machine learning accelerators 
	* CoreML
	* Accelerate, Compression, and simd
* Asymmetric processing (AMP).
	* Set your QoS.  Factor in determining which core a task will be run on.
	* Use GCD.  `concurrentPerform(iterations: execute:)`.
	* Make sure you're breaking your tasks over a large enough number of iterations.

[[Direct access to Video Encoding and Decoding - 14]]
[[ProRes Decoding with AVFoundation and VideoToolbox]]
[[Introducing Core ML - 17]]
[[Core ML 3 Framework - 19]]
[[Building Responsive and Efficient Apps with GCD - 15]]
[[Modernizing Grand Central Dispatch Usage]]
[[Using Accelerate and simd - 18]]
[[Introducing Accelerate for Swift - 19]]

[[Bring your  metal app to apple silicon]]
[[Optimize metal performance for apple silicon macs]]

# Security
* W^X
* Kernel Integrity Protection
* Pointer authentication
* Device isolation

## W ^ X
Memory pages cannot be both writable and executble at the same time.
JIT compilers
* write instructions into RWX memory
* execute instructions in RWX memory

`pthread_jit_write_protect_np()`
* fast switch between RW and RX permissions
* per-thread to support multithreaded JITs
* different threads can see different permissions of the same page

## KIP
* Hardware enforced kernel immutability
* When the kernel is initialized
	* No modification
	* No new code loaded

## Pointer authentication
* Pointers signed to guard against misuse
* Pointer authentication code (PAC) stored in unused high bits of pointer
* enabled for
	* kernel
	* system applications
	* system services

We're not yet ready for you to use this, but if you'd liek to experiment

`sudo nvram boot-args=-arm64e_preview_abi`
Add arm64e to the "architecutres" build setting in xcode
developer.apple.com/documentation/security/preparing_your_app_to_work_with_pointer_authentication

## Device isolation
Intel-based macs use a shared IOMMU.  
On apple silicon, each device uses its own IOMMU.  Devices can't snoop on each other.

```cpp
// Get the IOMapper for the device
IOMapper *mapper = IOMapper::copyMapperForDevice(device);

// Use an IODMACommand; pass the mapper when initializing
IODMACommand *dmaCommand = IODMACommand::withSpecification(
   outSegFunc, numAddressBits, maxSegmentSize, mappingOptions,
   maxTransferSize, alignment, mapper, refCon);

// Keep the IODMACommand prepared for the duration of the i/o
```

## Kernel extensions
Now require a reboot.
You should expect to see *more friction* around kexts.

## DriverKit
[[System Extensions and DriverKit - 19]]
[[Modernize PCI and SCSI drivers with DriverKit]]

# Rosetta
Rosetta runs
* macOS applications
* Catalyst applications
* games
* web browsers
* JIT compilers
* Metal directly on the Apple GPU
* CoreML with neural engine

Whole application compilation
* app store installation
* package installation
* first app launch

translations are code signed
and refreshed on OS update

Emulates x86_64 syscall interface
JIT translation of new code
Hardened runtime
supports xcode and instruments

High fidelity to x86_64 behavior
* 4KB pages
* TSO memory ordering
* 1GHz `mach_absolute_time`
* Floating point NaN, denormal handling

Check before using AVX
* `hw.optional.avx1_0`

DTK has some additional restrictions, see release notes.


```c
// Use "sysctl.proc_translated" to check if running in Rosetta

// Returns 1 if running in Rosetta
int processIsTranslated() {
   int ret = 0;
   size_t size = sizeof(ret);

   // Call the sysctl and if successful return the result
   if (sysctlbyname("sysctl.proc_translated", &ret, &size, NULL, 0) != -1) 
      return ret;

   // If "sysctl.proc_translated" is not present then must be native
   if (errno == ENOENT)
      return 0;

   return -1;
}
```

## Native applications
Port your mac applications
* developer.apple.com/documentation/apple_silicon/
* iPad and iPhone apps

[[Port your mac app to apple silicon]]
[[IPad and iPhone Apps on Apple Silicon Macs]]

# Boot architecture
## Boot overview
Based on iOS< iPadOS secure boot
enhanced to support
* multiple macOS installs
* Multiple versions of macOS
* macOS recovery flows



## Start-up and macOS recovery

### Start-up experience
Press and hold touchID or power to launch startup options
Existing startup keys are replaced by UI interactions

### macOS recovery
* startup UI with integrated startup manager
* startup disk
* mac sharing mode

### mac sharing mode

Replaces target disk mode
SMB file share based
User authentication required

### startup disk
Focuses on selecting security policy for each of the volume
choose from full or reduced security

* Full security: best-in-class security like iPhone, external disk boot without downgrading security
* Reduced security provides flexibility and configureability of your mac.  Any version of macOS, including versions no longer signed by apple.  Notarized 3rd-party kernel extensions.  

## Boot security

Configureable security
Available using `csrutil`
* secure boot
* root volume authentication
* system integrity protection

On Intel-based macs, security policy applies to the entire system.  So downgrading security will downgrade all macOS installed.

Apple silicon maintain a separate security policy for each macOS installation.  So you can downgrade security for one OS and keep another one intact.

## Login/data protection

Unified experience with and without FileVault
* Rich UI with accelerated graphics
* Fully boot macOS without requiring the user to unlock the system.

SmartCard support
* CCID and PIV
* VoiceOver support

Data protection
* Always-on volume encryption.  When filevault is on, this is tied to suer's credentials

Secure hibernation
* Full at-rest protection
* Integrity and anti-replay protection

Recoverying your mac
* macOS and macOS recovery

If macOS is not available, use macOS recovery to reinstall macOS.

What happens without macOS recovery?  On intel-based macs, you can use internet recovery.

On apple silicon macs, we have System Recovery.
* minimal macOS
* separate hidden container
* Let you re-install macOS and macOS recovery.

Apple Configurator 2
* Erase and install macOS *including* system recovery.
