#accelerate

# Accelerate framework
High-performance, energy-efficient vectorized computations
* macOS
* iOS
* iPadOS
* watchOS
* tvOS

Access to ML accelerators in apple silicon.  Only through accelerate.  (Or with secret instructions...)

* vDSP - signal processing
* vImage - image processing
* vForce - vector transcendental functions
* BLAS, LAPACK - dense matrix computations
* Sparse BLAS, sparse solvers - sparse matrix
* BNNS - ML

## Related frameworks
simd.h - small vector and matrix computations
Compression, AppleArchive - lossless data compression

```
// How to use in your own code

// C / Objective C / C++
#include <Accelerate/Accelerate.h> // Add framework Accelerate
#include <AppleArchive/AppleArchive.h> // Add framework AppleArchive
#include <Compression/Compression.h> // Add framework Compression
#include <simd/simd.h> // No framework to add

// Swift
import Accelerate
import AppleArchive
import Compression
import simd
```

# What's new in BNNS
Basic Neural Network Subroutines
Perf primitives for CPU ML

* CPU  - BNNS
* GPU - MPS
* Neural Engine

Number of frameworks that run on these backends
* CoreML
* CreateML
* Vision
* Natural Language
* ?

Several new layer types
* Embedding
* Random Fill
* Quantization
* AdamW optimizer

New activations
* SiLU
* HardSwish

New arithmetic functions

## New layer fusions
Convolution -> Quantization
etc.

## Other improvements
New clipping in optimizer
Standarone functions
?




# Improvements to simd.h
Small vectors, fit into registers
* Arithmetic operators
* geometry
* transcendental
* quaternions

90% of the benefit of vectorization with 10% of effort.

```swift
import simd

public func swishharder_scalar (_ data: [Float]) -> [Float] {
  return data.map { x in
    if x <= -.pi { return 0 }                         //        {  0                if  x ≤ -π
    if x <=  .pi { return 2*exp(x) * (x + .pi)/2 }    // f(x) = {  2eˣ * (x+π)/2    if -π <  x < π
    else         { return 2*exp(x) }                  //        {  2eˣ              if  π ≤  x
  }
}
```
But how to make it faster?
Mostly we use `replacing` to implement the stepwise fn.
```swift
func swishharder_elementwise(_ x: SIMD8<Float>) -> SIMD8<Float> {
    let y = 2*simd.exp(x)
    let a = y.replacing(with: 0, where: x .<= -.pi)
    let b = ((x + .pi)/2).replacing(with: 1, where: .pi .<= x)
    return a*b
}

extension SIMD {
    internal func store(
        into buffer: UnsafeMutableBufferPointer<Scalar>,
        startingAt offset: Int
    ) {
        for i in 0 ..< scalarCount {
            buffer[offset + i] = self[i]
        }
    }
}

public func swishharder_simd(_ data: [Float]) -> [Float] {
    return Array<Float>(unsafeUninitializedCapacity: data.count) {
        (buffer: inout UnsafeMutableBufferPointer<Float>, count: inout Int) in
        for i in stride(from: 0, to: data.count, by: 8) {
            let v = SIMD8(data[i ..< i+8])
            let w = swishharder_elementwise(v)
            w.store(into: buffer, startingAt: i)
        }
        count = data.count
    }
}
```

speedup 2.87x.

* Improved C++ usability
* TEmplated data type
* Query type traits

Aliases to simplify coding 

```cpp
typealias simd::Vector<float, 4>::type v1
```

etc.

Templated make/convert functions.  

## New simd functions

* Classification functions 
	* `isfinite` `isinf`, etc. that provide vector versions of these
* `lgamma`
* `simd_trace` "trace of simd matrices"
# Apple Encrypted Archive

* Powering system updates for almost 10 years
* Three complementary parts
	* Archive format (11 big sur)
	* compressed archive (11 big sur)
	* encryption support (new)

* Modular
	* Select included fields
	* Add new fields
	* Metadata entries

Streamable (e.g. don't fit data in memory)
manifests 
* Archive with no file data
* indices to the main archive

State of the art crypto
Data confidentiality
Data authenticity
Sender authentication
Signature privacy
Metadata protection
Re-signing attack protection

To facilitate, we offer different profiles for use cases

## Profiles
No encryption, signed.  e.g. software updates.
Symmetric key encryption, optionally signed
Password encryption
Public key encryption, optionally signed

compression is optional
data is authenticated

## CLI
`compression_tool` for compressed archive
`aea` => encrypted archive

API
* AppleArchive Swift and C
* Stream-based
* Sequential/random access
* Multi-threaded

1.  Encryption context
2.  File stream
3.  Encryption stream
4.  Encoder stream

Note we feed data in reverse
1.  encoder stream
2.  encrpytion stream
3.  file stream

```swift
/// Encrypts and archives the directory at the specified URL.
    ///
    /// - Parameter sourceURL: The URL of the directory that the function encrypts and archives.
    /// - Returns: A string containing the status message.
    ///
    /// This function writes the result to `directory`. The archive shares the name
    /// of the supplied directory with an `aea` extension.
    static func encrypt(sourceURL: URL) -> ConsoleMessage {
        
        // Verify that the URL path represents a directory.
        if !sourceURL.hasDirectoryPath {
            return ConsoleMessage(status: .error,
                                  message: "The specified URL doesn't point to a directory.")
        }

        guard let source = FilePath(sourceURL) else {
            return ConsoleMessage(status: .error,
                                  message: "Unable to create file path from source URL.")
        }

        // Create the destination `FilePath`.
        let destination = directory
            .appending(sourceURL.lastPathComponent)
            .appending(".aea")
        let archiveDestination = FilePath(destination)

        // Create the encryption context + setup credentials
        let encryptionKey = SymmetricKey(size: .bits256)

        let context = ArchiveEncryptionContext(
            profile: .hkdf_sha256_aesctr_hmac__symmetric__none,
            compressionAlgorithm: .lzfse)
        do {
            try context.setSymmetricKey(encryptionKey)
        } catch {
            return ConsoleMessage(status: .error,
                                  message: "Error setting password (\(error).)")
        }

        // Create the file stream, encryption stream, archive encode stream.
        guard
            let archiveDestinationFileStream = ArchiveByteStream.fileStream(
                path: archiveDestination,
                mode: .writeOnly,
                options: [ .create, .truncate ],
                permissions: FilePermissions(rawValue: 0o644)),
            
                let encryptionStream = ArchiveByteStream.encryptionStream(
                    writingTo: archiveDestinationFileStream,
                    encryptionContext: context),
            
                let encoderStream = ArchiveStream.encodeStream(
                    writingTo: encryptionStream) else {
                    
                    return ConsoleMessage(status: .error,
                                          message: "Error creating streams.")
                }
        
        // Remember to close things in the correct order
        defer {
            try? encoderStream.close()
			//in partcicular, this one is a lot of work to close since it signs and seals.
            try? encryptionStream.close()
            try? archiveDestinationFileStream.close()
        }

        // Encode all files in the target directory with the specifed fields.
        do {
            
            let fields = ArchiveHeader.FieldKeySet("TYP,PAT,DAT,UID,GID,MOD")!
            try encoderStream.writeDirectoryContents(archiveFrom: source,
                                                     keySet: fields)
        } catch {
            return ConsoleMessage(status: .error,
                                  message: "Error writing directory contents.")
        }
        
        if let url = URL(archiveDestination) {
            NSWorkspace.shared.activateFileViewerSelecting([url])
        }

        let message = """
Encrypted with key '\(encryptionKey.base64Encoded)'.
Archived and encrypted to: \(archiveDestination.description).
"""
        
        return ConsoleMessage(status: .encryptionSuccess(base64EncodedKeyData: encryptionKey.base64Encoded),
                              message: message)
    }
```

# Wrap up
* Optimized computation across all apple paltforms
* new layer types and improved performance for BNNS
* Expanded C++ support in simd.h
* Apple archive: high performance archive, encryption, compression

