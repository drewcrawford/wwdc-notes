#metal 

simplify/optimize resource loading for games/apps
# Introduction
build low latency asset loading pipelines
reduce load times by optimally utilizing faster storage
stream higher quality assets with fine-grained loading

dynamically stream objects.  Game only loads what it needs at first and gradually streams other resources as the player moves through the level.  Basically mipmapping?

Reduce streaming granularity.  Maybe only visible regions?  Sparse textures?  etc.  Modern hardware can do multiple load requests at once.

Can prioritze requests to ensure that high-priority requests finish in time.
# Key features
How FRL helps you.
* asynchronously load metal resources
* Maximize throughput with concurrent loads
* Batch loads to reduce overhead
* Prioritize operations for low latency

## steps
* open a file

```swift
// Create an Metal File IO Handle

// Create handle to an existing file
var fileIOHandle: MTLIOFileHandle!
do {
    try fileHandle = device.makeIOHandle(url: filePath)
} catch {
    print(error)
}
```

* issue load commands

don't wait for commands to finish.  IO buffers are concurrent etc.  This execution model better utilizes faster storage hardware by maximizing throughput.
3 types of commands
* texture
* buffer
* bytes (CPU accessible memory)

```swift
// Create a Metal IO command queue

let commandQueueDescriptor = MTLIOCommandQueueDescriptor()

commandQueueDescriptor.type = MTLIOCommandQueueType.concurrent // or serial

var ioCommandQueue: MTLIOCommandQueue!
do {
    try ioCommandQueue = device.makeIOCommandQueue(descriptor: commandQueueDescriptor)
} catch {
    print(error)
}
// Create Metal IO Command Buffer

let ioCommandBuffer = ioCommandQueue.makeCommandBuffer()

// Encode load commands
// Encode load texture and load buffer commands
ioCommandBuffer.load(texture, slice: 0, level: 0, size: size, 
                     sourceBytesPerRow:bytesPerRow, sourceBytesPerImage: bytesPerImage,  
                     destinationOrigin: destOrigin,
                     sourceHandle: fileHandle, sourceHandleOffset: 0)
ioCommandBuffer.load(buffer, offset: 0, size: size,
                     sourceHandle: fileHandle, sourceHandleOffset: 0)


// Commit command buffer for execution
ioCommandBuffer.commit()
```

* synchronize loading and rendering

```swift
var sharedEvent: MTLSharedEvent!
sharedEvent = device.makeSharedEvent()

// Create Metal IO command buffer
let ioCommandBuffer = ioCommandQueue.makeCommandBuffer()

ioCommandBuffer.waitForEvent(sharedEvent, value: waitVal)

// Encode load commands

ioCommandBuffer.signalEvent(sharedEvent, value: signalVal)
ioCommandBuffer.commit()

// Graphics work waits for the IO command buffer to signal
```

# Advanced features
## cancel
```swift
// Player in the center region
// Encode loads for the North-West region
ioCommandBufferNW.commit()
// Encode loads for the West region
ioCommandBufferW.commit()
// Encode loads for the South-West region
ioCommandBufferSW.commit()

// Player in the western region and heading south
// tryCancel NW command buffer
ioCommandBufferNW.tryCancel()

// ..
// ..
func regionNWCancelled() -> Bool {
    return ioCommandBufferNW.status == MTLIOStatus.cancelled
}
```

## prioritize
by queue?

```swift
// Create a Metal IO command queue

let commandQueueDescriptor = MTLIOCommandQueueDescriptor()
commandQueueDescriptor.type = MTLIOCommandQueueType.concurrent // or serial

// Set Metal IO command queue Priority
commandQueueDescriptor.priority = MTLIOPriority.high // or normal or low

var ioCommandQueue: MTLIOCommandQueue!
do {
    try ioCommandQueue = device.makeIOCommandQueue(descriptor: commandQueueDescriptor)
} catch {
    print(error)
}
```

priorities cannot be changed after creation.

# Best practices
* Reduce disk footprint with compression
* Improve throughput by tuning sparse page size

## Compression
`MTLIOCompressionContext`.  

```swift
// Create a compressed file

// Create compression context
let chunkSize = 64 * 1024
let compressionMethod = MTLIOCompressionMethod.zlib
let compressionContext = MTLIOCreateCompressionContext(compressedFilePath, compressionMethod, chunkSize)

// Append uncompressed file data to the compression context
// Get uncompressed file data 
MTLIOCompressionContextAppendData(compressionContext, filedata.bytes, filedata.length)


// Write the compressed file
MTLIOFlushAndDestroyCompressionContext(compressionContext)
```

metal 3 does inline decompression.  

```swift
// Create an Metal File IO Handle

// Create handle to a compressed file
var compressedFileIOHandle : MTLIOFileHandle!
do {
    try compressedFileHandle = device.makeIOHandle(url: compressedFilePath, compressionMethod: MTLIOCompressionMethod.zlib)
} catch {
    print(error)
}
```

compression methods:
| method   |                                                    |
| -------- | -------------------------------------------------- |
| lz4      | high encode and decode performance                 |
| zlib     | balanced and cross-platform                        |
| lzbitmap | balanced with higher encode and decode performance |
| lzfse    | balanced with higher cmpression ratio              |
| lzma     | best compression ratio                             |

custom compression scheme.  Can replace context with your own compressor.

## sparse tile size
16kb granularity.  In metal3 can specify 64 and 256kb.  Stream at larger granularity.

tradeoff between larger tile sizes etc.  Experiment to see which sizes work best.

# Tools
xc14 has full support for frl.
* runtime profiling
	* [[Optimize Metal apps and games with GPU counters]]
	* [[Optimize high-end games for Apple GPUs]]
* api inspection
	* frame capture, etc.
* dependency analysis
	* Dependency viewer.  I think this is somewhere in GPU capture.
	* [[Go bindless with Metal 3]]
# Example
# Wrap up
* utilize faster storage hardware
* reduce load times and improve image quality
* synchronize asest loading with graphics and compute
* minimize asset streaming latency

