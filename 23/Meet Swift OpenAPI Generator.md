Discover how Swift OpenAPI Generator can help you work with HTTP server APIs whether you're extending an iOS app or writing a server in Swift. We'll show you how this package plugin can streamline your workflow and simplify your codebase by generating code from an OpenAPI document.

New swift package plugin can streamline your workflow and simplify your codebase.  Easier to work with data on device.  Sometimes the feature you want is provided by a server.  Making network requests, etc.

1.  baseURL
2. path components
3. HTTP method to use
4. How to provide parameters
5. etc

## Reasoning about API behavior
* documentation.  But is it outdated?
* inspection, experiment?  incomplete understanding?
* discussion.  But inconsistent answers?

Using a more formal and structured description can eliminate ambiguity

# Exploring OpenAPI
tried and tested industry standard
established conventions and best practices
declare your API in YAML or JSON
rich ecosystem of tools

interactive documentation.  Core motivation is code generation, which allows to use... 

code we need to write, without using code generation.

## working with larger APIS
* handwritten code is fine for simple operations
* real-world apis can have many operations
* repetitive, verbose, error prone
* detracts from the core logic of your app

Use tooling to generate this code.

### Example OpenAPI document - 4:17
```yaml
openapi: "3.0.3"
info:
  title: "GreetingService"
  version: "1.0.0"
servers:
- url: "http://localhost:8080/api"
  description: "Production"
paths:
  /greet:
    get:
      operationId: "getGreeting"
      parameters:
      - name: "name"
        required: false
        in: "query"
        description: "Personalizes the greeting."
        schema:
          type: "string"
      responses:
        "200":
          description: "Returns a greeting"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Greeting"
```

If the operation accepts parameters, they can be included.

reduce ceremony using code generation.

* swift package plugin
* code generated at build time
* always in sync with the openapi document
* no need to commit to source control
[[Meet Swift Package Plugins]]

# Make API calls from your app

* add swift OpenAPI Generator dependencies
* configure target to us e the plugin
* add OpenAPI document and plugin config file
* replace UI components
* use generated Client type

need to depend on `swift-openapi-generator` and `swift-openapi-runtime`.
and an integration package `swift-openapi-urlsession`.  Check out the documentation for other examples.

"run build tool plugins" and choose "OpenAPIGenerator".  

plug


### CatService openapi.yaml - 7:05
```yaml
openapi: "3.0.3"
info:
  title: CatService
  version: 1.0.0
servers:
  - url: http://localhost:8080/api
    description: "Localhost cats 🙀"
paths:
  /emoji:
    get:
      operationId: getEmoji
      parameters:
      - name: count
        required: false
        in: query
        description: "The number of cats to return. 😽😽😽"
        schema:
          type: integer
      responses:
        '200':
          description: "Returns a random emoji, of a cat, ofc! 😻"
          content:
            text/plain:
              schema:
                type: string
```

### Making API calls from your app - 8:03
```swift
import SwiftUI
import OpenAPIRuntime
import OpenAPIURLSession

#Preview {
    ContentView()
}

struct ContentView: View {
    @State private var emoji = "🫥"

    var body: some View {
        VStack {
            Text(emoji).font(.system(size: 100))
            Button("Get cat!") {
                Task { try? await updateEmoji() }
            }
        }
        .padding()
        .buttonStyle(.borderedProminent)
    }

    let client: Client

    init() {
        self.client = Client(
            serverURL: try! Servers.server1(),
            transport: URLSessionTransport()
        )
    }

    func updateEmoji() async throws {
        let response = try await client.getEmoji(Operations.getEmoji.Input())

        switch response {
        case let .ok(okResponse):
            switch okResponse.body {
            case .text(let text):
                emoji = text
            }
        case .undocumented(statusCode: let statusCode, _):
            print("cat-astrophe: \(statusCode)")
            emoji = "🙉"
        }
    }
}
```

### CatServiceClient openapi-generator-config.yaml - 9:48
```yaml
generate:
  - types
  - client
```

As a security measure, you'll be asked to trust the plugin on first use.

# Adapting as the API evolves
Let's walk through an example.

1.  Add parameter to the OpenAPI document
2. recompile the project to regenerate the types
3. create another button, and use the new parameter.


### Adapting as the API evolves - 13:24
```swift
import SwiftUI
import OpenAPIRuntime
import OpenAPIURLSession

#Preview {
    ContentView()
}

struct ContentView: View {
    @State private var emoji = "🫥"

    var body: some View {
        VStack {
            Text(emoji).font(.system(size: 100))
            Button("Get cat!") {
                Task { try? await updateEmoji() }
            }
            Button("More cats!") {
                Task { try? await updateEmoji(count: 3) }
            }
        }
        .padding()
        .buttonStyle(.borderedProminent)
    }

    let client: Client

    init() {
        self.client = Client(
            serverURL: try! Servers.server1(),
            transport: URLSessionTransport()
        )
    }

    func updateEmoji(count: Int = 1) async throws {
        let response = try await client.getEmoji(Operations.getEmoji.Input(
            query: Operations.getEmoji.Input.Query(count: count)
        ))

        switch response {
        case let .ok(okResponse):
            switch okResponse.body {
            case .text(let text):
                emoji = text
            }
        case .undocumented(statusCode: let statusCode, _):
            print("cat-astrophe: \(statusCode)")
            emoji = "🙉"
        }
    }
}
```

# Testing your app with mocks
Because client type conforms to a swift protocol, easy to mock.
`APIProtocol`.  Define a new `MockClient` that adopts the protoco.l
make view geenric over `C:APIProtocol`
Update initializers to support dependency injection
Use `MockClient` in the Xcode preview


### Testing your app with mocks - 15:06
```swift
import SwiftUI
import OpenAPIRuntime
import OpenAPIURLSession

#Preview {
    ContentView(client: MockClient())
}

struct ContentView<C: APIProtocol>: View {
    @State private var emoji = "🫥"

    var body: some View {
        VStack {
            Text(emoji).font(.system(size: 100))
            Button("Get cat!") {
                Task { try? await updateEmoji() }
            }
            Button("More cats!") {
                Task { try? await updateEmoji(count: 3) }
            }
        }
        .padding()
        .buttonStyle(.borderedProminent)
    }

    let client: C

    init(client: C) {
        self.client = client
    }

    init() where C == Client {
        self.client = Client(
            serverURL: try! Servers.server1(),
            transport: URLSessionTransport()
        )
    }

    func updateEmoji(count: Int = 1) async throws {
        let response = try await client.getEmoji(Operations.getEmoji.Input(
            query: Operations.getEmoji.Input.Query(count: count)
        ))

        switch response {
        case let .ok(okResponse):
            switch okResponse.body {
            case .text(let text):
                emoji = text
            }
        case .undocumented(statusCode: let statusCode, _):
            print("cat-astrophe: \(statusCode)")
            emoji = "🙉"
        }
    }
}

struct MockClient: APIProtocol {
    func getEmoji(_ input: Operations.getEmoji.Input) async throws -> Operations.getEmoji.Output {
        let count = input.query.count ?? 1
        let emojis = String(repeating: "🤖", count: count)
        return .ok(Operations.getEmoji.Output.Ok(
            body: .text(emojis)
        ))
    }
}
```

# Server development in Swift
* swift package using Swift OpenAPI Generator
* Defines a type conforming to APIProtocol
* Uses registerHandlers to configure the server

Here we started with the OpenAPI and used the API generator to write the server.

In this demo, we're using Vapour.  But the generator code can be used with any web framework.  Check out the documentation for other options and how you can write your own.


### Implementing a backend server - 16:58
```swift
import Foundation
import OpenAPIRuntime
import OpenAPIVapor
import Vapor

struct Handler: APIProtocol {
    func getEmoji(_ input: Operations.getEmoji.Input) async throws -> Operations.getEmoji.Output {
        let candidates = "🐱😹😻🙀😿😽😸😺😾😼"
        let chosen = String(candidates.randomElement()!)
        let count = input.query.count ?? 1
        let emojis = String(repeating: chosen, count: count)
        return .ok(Operations.getEmoji.Output.Ok(body: .text(emojis)))
    }
}

@main
struct CatService {
    public static func main() throws {
        let app = Vapor.Application()
        let transport = VaporTransport(routesBuilder: app)
        let handler = Handler()
        try handler.registerHandlers(on: transport, serverURL: Servers.server1())
        try app.run()
    }
}
```

To learn more about writing backend services in swift, see [[Use Xcode for server-side development]]


### CatService Package.swift - 18:43
```swift
// swift-tools-version: 5.8
import PackageDescription

let package = Package(
    name: "CatService",
    platforms: [
        .macOS(.v13),
    ],
    dependencies: [
        .package(
             url: "https://github.com/apple/swift-openapi-generator",
            .upToNextMinor(from: "0.1.0")
        ),
        .package(
            url: "https://github.com/apple/swift-openapi-runtime",
            .upToNextMinor(from: "0.1.0")
        ),
        .package(
            url: "https://github.com/swift-server/swift-openapi-vapor",
            .upToNextMinor(from: "0.1.0")
        ),
        .package(
            url: "https://github.com/vapor/vapor",
            .upToNextMajor(from: "4.69.2")
        ),
    ],
    targets: [
        .executableTarget(
            name: "CatService",
            dependencies: [
                .product(name: "OpenAPIRuntime", package: "swift-openapi-runtime"),
                .product(name: "OpenAPIVapor", package: "swift-openapi-vapor"),
                .product(name: "Vapor", package: "vapor"),
            ],
            resources: [
                .process("Resources/cat.mp4"),
            ],
            plugins: [
                .plugin(name: "OpenAPIGenerator", package: "swift-openapi-generator"),
            ]
        ),
    ]
)
```


### CatService openapi.yaml - 19:08
```yaml
openapi: "3.0.3"
info:
  title: CatService
  version: 1.0.0
servers:
  - url: http://localhost:8080/api
    description: "Localhost cats 🙀"
paths:
  /emoji:
    get:
      operationId: getEmoji
      parameters:
      - name: count
        required: false
        in: query
        description: "The number of cats to return. 😽😽😽"
        schema:
          type: integer
      responses:
        '200':
          description: "Returns a random emoji, of a cat, ofc! 😻"
          content:
            text/plain:
              schema:
                type: string
```

### CatService openapi-generator-config.yaml - 19:10
```yaml
generate:
  - types
  - server
```
1.  Add new operation to the OpenAPI document
2. add new APIProtocol function to Handler

### Adding an operation to the OpenAPI document - 20:11
```yaml
openapi: "3.03"
info:
  title: CatService
  version: 1.0.0
servers:
  - url: http://localhost:8080/api
    description: "Localhost cats 🙀"
paths:
  /emoji:
    get:
      operationId: getEmoji
      parameters:
      - name: count
        required: false
        in: query
        description: "The number of cats to return. 😽😽😽"
        schema:
          type: integer
      responses:
        '200':
          description: "Returns a random emoji, of a cat, ofc! 😻"
          content:
            text/plain:
              schema:
                type: string

  /clip:
    get:
      operationId: getClip
      responses:
        '200':
          description: "Returns a cat video! 😽"
          content:
            video/mp4:
              schema:
                type: string
                format: binary
```

### Adding a new API operation - 20:26
```swift
import Foundation
import OpenAPIRuntime
import OpenAPIVapor
import Vapor

struct Handler: APIProtocol {
    func getClip(_ input: Operations.getClip.Input) async throws -> Operations.getClip.Output {
        let clipResourceURL = Bundle.module.url(forResource: "cat", withExtension: "mp4")!
        let clipData = try Data(contentsOf: clipResourceURL)
        return .ok(Operations.getClip.Output.Ok(body: .binary(clipData)))
    }
    
    func getEmoji(_ input: Operations.getEmoji.Input) async throws -> Operations.getEmoji.Output {
        let candidates = "🐱😹😻🙀😿😽😸😺😾😼"
        let chosen = String(candidates.randomElement()!)
        let count = input.query.count ?? 1
        let emojis = String(repeating: chosen, count: count)
        return .ok(Operations.getEmoji.Output.Ok(body: .text(emojis)))
    }
}

@main
struct CatService {
    public static func main() throws {
        let app = Vapor.Application()
        let transport = VaporTransport(routesBuilder: app)
        let handler = Handler()
        try handler.registerHandlers(on: transport, serverURL: Servers.server1())
        try app.run()
    }
}
```

# Wrap up
* document your service with OpenAPI
* streamline your workflow with Swift OpenAPI generator
* Explore server development in Swift

# Resources
* https://github.com/apple/swift-openapi-generator
* http://github.com/apple/swift-openapi-runtime
* https://github.com/apple/swift-openapi-urlsession
