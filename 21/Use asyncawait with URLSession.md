#async 

[[Meet asyncawait in Swift]]

* Linear
* concise
* Native error handling

>  We have to use extreme caution towards data races.

completion handler are not consistently dispatched to the main queue.

Early return, completion handler can be called twice.

UIImage creation can fail.  So we would have called completion handler with `nil` image and `nil` error.

```swift
// Fetch photo with async/await

func fetchPhoto(url: URL) async throws -> UIImage
{
    let (data, response) = try await URLSession.shared.data(from: url)

    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw WoofError.invalidServerResponse
    }

    guard let image = UIImage(data: data) else {
        throw WoofError.unsupportedImage
    }

    return image
}
```

```swift
let (data, response) = try await URLSession.shared.data(from: url)
guard let httpResponse = response as? HTTPURLResponse,
      httpResponse.statusCode == 200 /* OK */ else {
    throw MyNetworkingError.invalidServerResponse
}
```

Accept either URL or request.  

Also provide upload

```swift
var request = URLRequest(url: url)
request.httpMethod = "POST"

let (data, response) = try await URLSession.shared.upload(for: request, fromFile: fileURL)
guard let httpResponse = response as? HTTPURLResponse,
      httpResponse.statusCode == 201 /* Created */ else {
    throw MyNetworkingError.invalidServerResponse
}
```

Note that `get` does not support uploads.

# Downloads

```swift
let (location, response) = try await URLSession.shared.download(from: url)
guard let httpResponse = response as? HTTPURLResponse,
      httpResponse.statusCode == 200 /* OK */ else {
    throw MyNetworkingError.invalidServerResponse
}

try FileManager.default.moveItem(at: location, to: newLocation)
```

These *will not delete the file*, do not forget to do it yourself!


```swift
let handle = async {
    let (data1, response1) = try await URLSession.shared.data(from: url1)

    let (data2, response2) = try await URLSession.shared.data(from: url2)

}

handle.cancel()
```

Later, we can use task handle to cancel.  Note that concurrency tasks are unrelated to `URLSession` tasks.

What about incremental?

# URLSession.bytes

Uses AsyncSequence

Parse each line of the response as json data.  

Enables us to consume the response line-by-lne as data is received.  

use `await` to call into the main thread.  

# AsyncSequence
* Built-in transformations
* System frameworks support

[[Meet AsyncSequence]]

# URLSessionTask-specific delegate

Now all methods can take `delegate` for a "task-specific delegate".  Provide an object specific to the operation

Also `NSURLSessionTask` has a `delegate` property.  Will be strongly held until task completes/fails

*Not supported by background URL session*.

This has a higher precedence than session delegate.  Evidently we only call 1.

Prompt the user for credentials.  Implement the URLSession `didReceiveChallenge` method.  Respond to HTTP basic challenges.

Delegate object is not an instance variable and is strongly-held by the task until the task completes or fails.

New, delegate can be used to handle events with specific task instance.  Handy when the logic only applies to certain tasks and not others.

# Wrap up
* Adopt URLSEssion `async` methods
* Apply async concepts to your code
	* completion handler => async function
	* events handler => async sequence

[[Analyze HTTP traffic in Instruments]]
[[Accelerate networking with HTTP3 and QUIC]]
