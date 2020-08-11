#serversideswift

# Extending applications into the cloud
Serverless functions offer a programming model that keeps the two closer together.

Traditional server deploment means dedicated compute available 24x7
Serverless Functions are becoming increasingly popular on the cloud
* simple scale-up and down
* control compute costs

AWS lambda
* among the industry leaders in this space
* supports multiple languages and runtimes

# Swift for serverless function
great for developers:
* safe
* expressive
* makes simple things easy

great for serverless application
* low memory footrprint
* deterministic performance
* quick start time

# swift on AWS lambda
```swift
//closure-based lambda function
import AWSLambdaRuntime

Lambda.run { (_, name: String, callback) in
    callback(.success("Hello, \(name)!"))
}
```

Closure will be invoked when things become available to process.  Process lifecycle is managed by the server situation

Protocol-oriented APIs for performance-sensitive usecases.

```swift
import AWSLambdaRuntime
import NIO

struct Handler: EventLoopLambdaHandler {
    typealias In = String
    typealias Out = String

    func handle(context: Lambda.Context, event: String) -> EventLoopFuture<String> {
       context.eventLoop.makeSucceededFuture("Hello, \(event)!")
    }
}

Lambda.run(Handler())
```
Lambda function needs to take care to never block the event loop.  In most cases, we think this API isn't worth it.

Using `Codable`

```swift
import AWSLambdaRuntime

struct Request: Codable {
    let name: String
    let password: String
}

struct Response: Codable {
    let message: String
}

Lambda.run { (_, request: Request, callback) in
    callback(.success(Response(message: "Hello, \(request.name)!")))
}
```
In most cases, Lambda payloads are JSON-based.

# Demo
Build/debug a lambda function.

Lambda is a package-manager project with a dependency on the lambda runtime.

Request/response struct similar to the one we saw on the slide.  How do we tie these two things together?

Let's run our Lambda func and see what happens.  Error, connection refused.  We're not running on Lambda, we're running on Xcode.

Instead, use special simulator mode.  Edit scheme -> environment variable `LOCAL_LAMBDA_SERVER_ENABLED` -> `true`.  

Now we see we're listening on `localhost:7000`.  

Xcode manages 2 processes.  One for lambda and one for the app.
Lambda function ~6.3mb ram.

Breakpoint demo involving setting breakpoints on both client and server.

## Local debugging explanation
* xcode manages executable targets
* lambda function was made available through a mock server
* Client used HTTP calls using URL session
* Designed for debugging, use AWS API Gateway when deploying to production

# Deploy
`./scripts/deploy.sh`
This first, creates a docker images.  Appears to be based on the aws linux image.  this installs... zip for some reason?  
Then we compile our program and upload it to aws.
`./scripts/test.sh` and see what happens

# How does it work?
* AWS manages the lambda function
	* compute resources will be hibernated until work is available.
* AWS API Gateway exposes the aLambda function as an HTTP endpoint.  Can be accessed by various clients
* Other ways to invoke lambda functions, such as event-based triggers.  See documentation.

# How does it work, more?
* AWS lambda uses Amazon Linux for its underlying virtual machine.
* swift.org began publishing toolchain for Amazon Linux in May 2020
* Toolchain can be used in the context of any compute service that uses Amazon Linux, including AWS lambda

## Runtime library
* an open source library designed to make building serverless functions in Swift simple and safe
* Multi-tier API for building wide range of serverless functions.

Program serves traffic forever.  Long-lived processes can serve faster, by employing caching techniques "such as caching the connection".  
The library also manages a state machine representing the various stages of the lamda thing.  Pulling work from the lambda queue, submitting work to the user-provided function, and submitting the results back ot the runtime engine.
A synchronous HTTP client that's optimized for this case is embedded in the library.
Compiling the package produces an executable that links this all together.  Can be linked statically or dynamically.

AWS Lambda can then be configured to run as many copies as required.  Serverless functions are designed to be stateless, and need to be careful not to hold onto state etc.

Finally, the runtime library includes extensible integration points to configure serverless functions base don events.

* Trigger serverless functions based on events from AWS systems like S3, SQS, SNS, and more
* Trigger serverless functions using HTTP endpoint exposed using AWS API gateway
* Extensible to additional event types


 
# Resources
https://github.com/swift-server/swift-aws-lambda-runtime/
https://swift.org/blog/aws-lambda-runtime/
