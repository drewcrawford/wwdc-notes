Find out how URLSession can help your apps transfer large files and recover from network interruptions. Learn how to pause and resume HTTP file transfers and support resumable uploads, and explore best practices for using URLSession to transfer files even when your app is suspended in the background.
#urlsession #swiftnio

how to deal with users who go out of wifi range?

# Pause and resume

Amazing.  But how does it work?

1.  Client sends GET
2. server responds `Accept-ranges: bytes`.
3. and ETag.
4. Client can send `If-range: ,etag>` and `Range:bytes=123-`.
5. Server can respond 206 Partial Content.
### 4:53 - Pausing and resuming a URLSessionDownloadTask
```swift
let downloadTask = session.downloadTask(with: request)
downloadTask.resume()
```

### 5:21 - Pausing and resuming a URLSessionDownloadTask
```swift
let downloadTask = session.downloadTask(with: request)
downloadTask.resume()

guard let resumeData = await downloadTask.cancelByProducingResumeData() else {
    // Download cannot be resumed
    return
}
```

the etag, etc., are stored in this `resumeData` object.  Important to note that this is not the partial download data.  If the resumeData is nil, one or more requirements for the resumable downloads are not met.

### 6:11 - Pausing and resuming a URLSessionDownloadTask
```swift
let downloadTask = session.downloadTask(with: request)
downloadTask.resume()

guard let resumeData = await downloadTask.cancelByProducingResumeData() else {
    // Download cannot be resumed
    return
}

let newDownloadTask = session.downloadTask(withResumeData: resumeData)
newDownloadTask.resume()
```

how to recover from unpredicted errors?

### 6:34 - Retrieving resume data on error
```swift
do {
    let (url, response) = try await session.download(for: request)
} catch let error as URLError {
    guard let resumeData = error.downloadTaskResumeData else {
        // Download cannot be resumed
        return
    }
}
```

* HTTPS or HTTP GET request
* Server supports byte-range
* Server provides ETag (or last-modified field).
* Temporary file still exists, not deleted due to disk space pressure.

iOS 17 has new support for resumable upload tasks.  Automatically resumable if the server supports the latest protocol draft.


### 8:29 - Pausing and resuming a URLSessionUploadTask
```swift
let uploadTask = session.uploadTask(with: request, fromFile: fileURL)
uploadTask.resume()
```

### 8:37 - Pausing and resuming a URLSessionUploadTask
```swift
let uploadTask = session.uploadTask(with: request, fromFile: fileURL)
uploadTask.resume()

guard let resumeData = await uploadTask.cancelByProducingResumeData() else {
    // Upload cannot be resumed
    return
}
```

### 8:57 - Pausing and resuming a URLSessionUploadTask
```swift
let uploadTask = session.uploadTask(with: request, fromFile: fileURL)
uploadTask.resume()

guard let resumeData = await uploadTask.cancelByProducingResumeData() else {
    // Upload cannot be resumed
    return
}

let newUploadTask = session.uploadTask(withResumeData: resumeData)
newUploadTask.resume()
```

If the interruption is silly and short, we will resume for you, don't worry about it.  But if it takes longer, we will give you the error.
### 9:22 - Retrieving resume data on error
```swift
do {
    let (data, response) = try await session.upload(for: request, fromFile: fileURL)
} catch let error as URLError {
    guard let resumeData = error.uploadTaskResumeData else {
        // Upload cannot be resumed
        return
    }
}
```

Your server must support the latest protocol.
* industry-wide collaboration
* draft-ietf-httpbis-resumable-upload-01
* automatic discovery
* transparent fallback

client hello
```http
POST /uploads
Upload-Incomplete: ?0
```
server reply
```http
104 Upload Resumption Supported
Location: /uploads/unique-resume-url
```

server finished
```http
201 created
```

if the upload is interrupted, we do the resumable upload procedures.  client needs to know how much data it got.

```http
HEAD /uploasd/unique-resume-url
```
server response
```http
204 No content
Upload-Offset: 1234
```

client says
```http
PATCH /uploads/unique-resume-url
Upload-Offset: 82898
{data }
```


# Build your server

### 13:15 - Before resumable uploads in Swift NIO
```swift
NIOTSListenerBootstrap(group: NIOTSEventLoopGroup())
    .childChannelInitializer { channel in
        channel.configureHTTP2Pipeline(mode: .server) { channel in
            channel.pipeline.addHandlers([
                HTTP2FramePayloadToHTTPServerCodec(),
                ExampleChannelHandler()
            ])
        }.map { _ in () }
    }
    .tlsOptions(tlsOptions)
```

this does bidirectional routing between HTTP and 'example channel'.

first we need NIOResumableUpload project.
We define the context.
### 14:06 - Add resumable uploads in Swift NIO
```swift
import NIOResumableUpload

let uploadContext = HTTPResumableUploadContext(origin: "https://example.com")

NIOTSListenerBootstrap(group: NIOTSEventLoopGroup())
    .childChannelInitializer { channel in
        channel.configureHTTP2Pipeline(mode: .server) { channel in
            channel.pipeline.addHandlers([
                HTTP2FramePayloadToHTTPServerCodec(),
                HTTPResumableUploadHandler(context: uploadContext, handlers: [
                    ExampleChannelHandler()
                ])
            ])
        }.map { _ in () }
    }
    .tlsOptions(tlsOptions)
```

### 15:48 - Informational responses in URLSession
```swift
protocol URLSessionTaskDelegate : URLSessionDelegate {
    optional func urlSession(_ session: URLSession, task: URLSessionTask,
                             didReceiveInformationalResponse response: HTTPURLResponse)
}
```

resumable protocols are great, etc.
# Go background
background URL session is also useful in handling large file transfers.  Imagine you're uploading a huge 4k video.  If their connection is interrupted, you want to resume if possible.

Let background session do it for you.
* automatic resumption
* automatic retry
* tasks will be scheduled after connecting to the internet again.
* out-of-process
* efficient use of resources


### 18:19 - Using background URLSession
```swift
// Configuring your background session
let configuration = URLSessionConfiguration.background(withIdentifier: "com.example.app")
configuration.isDiscretionary = true
configuration.allowsConstrainedNetworkAccess = false
let session = URLSession(configuration: configuration, delegate: self, delegateQueue: nil)

// Configuring your background task
let backgroundTask = session.uploadTask(with: url, fromFile: fileURL)
backgroundTask.earliestBeginDate = .now.addingTimeInterval(60 * 60)
backgroundTask.countOfBytesClientExpectsToSend = 500 * 1024
backgroundTask.countOfBytesClientExpectsToReceive = 200
```

[[Advances in networking, Pt 1]]

# Next steps
* adopt resumable uploads
* provide feedback
* Try background URLSession for large or discretionary transfers

 [[Reduce network delays with L4S]]
 [[Ready, set, relay Protect app traffic with network relays]]
 
# Resources
* https://developer.apple.com/documentation/foundation/urlsession/building_a_resumable_upload_server_with_swiftnio
* https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_in_the_background
* https://developer.apple.com/documentation/foundation/urlsession
