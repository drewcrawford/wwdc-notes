Swift is a great language for writing your server applications, and powers critical services across Apple's cloud products. We'll explore tooling, delve into the Swift server package ecosystem, and demonstrate how to interact with databases and add observability to applications.

# Meet swift on server
C-like performance
low memory footprint
safe and expressive
First-class concurrency features

An excellet choice for server applications.
* iCloud keychain
* photos
* notes
* app store processing pipelines
* shareplay sharing
* Private cloud compute (new)
Serving millions of requests per second in swift on server.

SSWG
* formed in 2016
* Consists of company representatives and individual contributors
* Promotes the use of swift for developing and deploying server aplpications.
* Defines and prioritzes the needs of the server community
* reduces duplication of efforst and increases compatibility
* channels feedback to other steering groups and workgroups

# build a service

1.  List all events
2. Create a new event

editors:
* xcode
* vscode
* neovim
* LSP supporting editors


### 3:23 - EventService Package.swift

```swift
// swift-tools-version:5.9
import PackageDescription

let package = Package(
    name: "EventService",
    platforms: [.macOS(.v14)],
    dependencies: [
        .package(
            url: "https://github.com/apple/swift-openapi-generator",
            from: "1.2.1"
        ),
        .package(
            url: "https://github.com/apple/swift-openapi-runtime",
            from: "1.4.0"
        ),
        .package(
            url: "https://github.com/vapor/vapor",
            from: "4.99.2"
        ),
        .package(
            url: "https://github.com/swift-server/swift-openapi-vapor",
            from: "1.0.1"
        ),
    ],
    targets: [
        .target(
            name: "EventAPI",
            dependencies: [
                .product(
                    name: "OpenAPIRuntime",
                    package: "swift-openapi-runtime"
                ),
            ],
            plugins: [
                .plugin(
                    name: "OpenAPIGenerator",
                    package: "swift-openapi-generator"
                )
            ]
        ),
        .executableTarget(
            name: "EventService",
            dependencies: [
                "EventAPI",
                .product(
                    name: "OpenAPIRuntime",
                    package: "swift-openapi-runtime"
                ),
                .product(
                    name: "OpenAPIVapor",
                    package: "swift-openapi-vapor"
                ),
                .product(
                    name: "Vapor",
                    package: "vapor"
                ),
            ]
        ),
    ]
)
```



# Explore the ecosystem

Swift OpenAPI generator is nice.

[[Meet Swift OpenAPI Generator]]

### 4:05 - EventService openapi.yaml

```yaml
openapi: "3.1.0"
info:
  title: "EventService"
  version: "1.0.0"
servers:
  - url: "https://localhost:8080/api"
    description: "Example service deployment."
paths:
  /events:
    get:
      operationId: "listEvents"
      responses:
        "200":
          description: "A success response with all events."
          content:
            application/json:
              schema:
                type: "array"
                items:
                  $ref: "#/components/schemas/Event"
    post:
      operationId: "createEvent"
      requestBody:
        description: "The event to create."
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Event'
      responses:
        '201':
          description: "A success indicating the event was created."
        '400':
          description: "A failure indicating the event wasn't created."
components:
  schemas:
    Event:
      type: "object"
      description: "An event."
      properties:
        name:
          type: "string"
          description: "The event's name."
        date:
          type: "string"
          format: "date"
          description: "The day of the event."
        attendee:
          type: "string"
          description: "The name of the person attending the event."
      required:
        - "name"
        - "date"
        - "attendee"
```


### 4:35 - EventService initial implementation

```swift
import OpenAPIRuntime
import OpenAPIVapor
import Vapor
import EventAPI

@main
struct Service {
    static func main() async throws {
        let application = try await Vapor.Application.make()
        let transport = VaporTransport(routesBuilder: application)
        let service = Service()
        
        try service.registerHandlers(
            on: transport,
            serverURL: URL(string: "/api")!
        )
        
        try await application.execute()
    }
}

extension Service: APIProtocol {
    func listEvents(
        _ input: Operations.listEvents.Input
    ) async throws -> Operations.listEvents.Output {
        let events: [Components.Schemas.Event] = [
            .init(name: "Server-Side Swift Conference", date: "26.09.2024", attendee: "Gus"),
            .init(name: "Oktoberfest", date: "21.09.2024", attendee: "Werner")
        ]
        
        return .ok(.init(body: .json(events)))
    }
    
    func createEvent(
        _ input: Operations.createEvent.Input
    ) async throws -> Operations.createEvent.Output {
        return .undocumented(statusCode: 501, .init())
    }
}
```

Database drivers.
* PostgreSQL
* MySQL
* cassandra
* many more!

PostgressNIO is maintained by Vapor and Apple

New is PostgresClient, new asynchronous interface
Robust and resilient connection handling
Higher throughput


Adding persistence
Add postgresNIO dependency
Use PostgresClient to query the database
Implement createEvent method


### 6:56 - EventService Package.swift

```swift
// swift-tools-version:5.9
import PackageDescription

let package = Package(
    name: "EventService",
    platforms: [.macOS(.v14)],
    dependencies: [
        .package(
            url: "https://github.com/apple/swift-openapi-generator",
            from: "1.2.1"
        ),
        .package(
            url: "https://github.com/apple/swift-openapi-runtime",
            from: "1.4.0"
        ),
        .package(
            url: "https://github.com/vapor/vapor",
            from: "4.99.2"
        ),
        .package(
            url: "https://github.com/swift-server/swift-openapi-vapor",
            from: "1.0.1"
        ),
        .package(
            url: "https://github.com/vapor/postgres-nio",
            from: "1.19.1"
        ),
    ],
    targets: [
        .target(
            name: "EventAPI",
            dependencies: [
                .product(
                    name: "OpenAPIRuntime",
                    package: "swift-openapi-runtime"
                ),
            ],
            plugins: [
                .plugin(
                    name: "OpenAPIGenerator",
                    package: "swift-openapi-generator"
                )
            ]
        ),
        .executableTarget(
            name: "EventService",
            dependencies: [
                "EventAPI",
                .product(
                    name: "OpenAPIRuntime",
                    package: "swift-openapi-runtime"
                ),
                .product(
                    name: "OpenAPIVapor",
                    package: "swift-openapi-vapor"
                ),
                .product(
                    name: "Vapor",
                    package: "vapor"
                ),
                .product(
                    name: "PostgresNIO",
                    package: "postgres-nio"
                ),
            ]
        ),
    ]
)
```

### 7:08 - Implementing the listEvents method

```swift
import OpenAPIRuntime
import OpenAPIVapor
import Vapor
import EventAPI
import PostgresNIO

@main
struct Service {
    let postgresClient: PostgresClient
    
    static func main() async throws {
        let application = try await Vapor.Application.make()
        let transport = VaporTransport(routesBuilder: application)
        let postgresClient = PostgresClient(
            configuration: .init(
                host: "localhost",
                username: "postgres",
                password: nil,
                database: nil,
                tls: .disable
            )
        )
        let service = Service(postgresClient: postgresClient)
        
        try service.registerHandlers(
            on: transport,
            serverURL: URL(string: "/api")!
        )
        
        try await withThrowingDiscardingTaskGroup { group in
            group.addTask {
                await postgresClient.run()
            }
            
            group.addTask {
                try await application.execute()
            }
        }
    }
}

extension Service: APIProtocol {
    func listEvents(
        _ input: Operations.listEvents.Input
    ) async throws -> Operations.listEvents.Output {
        let rows = try await self.postgresClient.query(
            "SELECT name, date, attendee FROM events"
        )
        
        var events = [Components.Schemas.Event]()
        
        for try await (name, date, attendee) in rows.decode((String, String, String).self) {
            events.append(
                .init(name: name, date: date, attendee: attendee)
            )
        }
        
        return .ok(.init(body: .json(events)))
    }
    
    func createEvent(
        _ input: Operations.createEvent.Input
    ) async throws -> Operations.createEvent.Output {
        return .undocumented(statusCode: 501, .init())
    }
}
```

### 9:02 - Implementing the createEvent method

```swift
func createEvent(
    _ input: Operations.createEvent.Input
) async throws -> Operations.createEvent.Output {
    switch input.body {
    case .json(let event):
        try await self.postgresClient.query(
            """
            INSERT INTO events (name, date, attendee)
            VALUES (\(event.name), \(event.date), \(event.attendee))
            """
        )
        return .created(.init())
    }
}
```

This transfers into parameterized query with value binding, so it's safe!  

PSQLError intentionally omits stuff so we don't leak data.

Observability has 3 pillars
* logging
* metrics
* tracing



### 11:34 - Instrumenting the listEvents method

```swift
func listEvents(
    _ input: Operations.listEvents.Input
) async throws -> Operations.listEvents.Output {
    let logger = Logger(label: "ListEvents")
    logger.info("Handling request", metadata: ["operation": "\(Operations.listEvents.id)"])
    Counter(label: "list.events.counter").increment()
    
    return try await withSpan("database query") { span in
        let rows = try await postgresClient.query(
            "SELECT name, date, attendee FROM events"
        )
        
        return try await .ok(.init(body: .json(decodeEvents(rows))))
    }
}
```

Can add metadata to log messages.
Add counter from swift-metrics to increment on each request. 
Add swift-distributed-tracing to create a span around each db query.
To learn more, chekc out

[[Beyond the basics of structured concurrency]]

We have many different backends for sending data.  Bootstrapping is only done in executables, and done as early as possible to ensure nothing is lost.

1.  First logging
2. then metrics
3. then instrumentation

Metrics/instrumentation might want to log!



### 13:14 - EventService Package.swift

```swift
// swift-tools-version:5.9
import PackageDescription

let package = Package(
    name: "EventService",
    platforms: [.macOS(.v14)],
    dependencies: [
        .package(
            url: "https://github.com/apple/swift-openapi-generator",
            from: "1.2.1"
        ),
        .package(
            url: "https://github.com/apple/swift-openapi-runtime",
            from: "1.4.0"
        ),
        .package(
            url: "https://github.com/vapor/vapor",
            from: "4.99.2"
        ),
        .package(
            url: "https://github.com/swift-server/swift-openapi-vapor",
            from: "1.0.1"
        ),
        .package(
            url: "https://github.com/vapor/postgres-nio",
            from: "1.19.1"
        ),
        .package(
            url: "https://github.com/apple/swift-log",
            from: "1.5.4"
        ),
    ],
    targets: [
        .target(
            name: "EventAPI",
            dependencies: [
                .product(
                    name: "OpenAPIRuntime",
                    package: "swift-openapi-runtime"
                ),
            ],
            plugins: [
                .plugin(
                    name: "OpenAPIGenerator",
                    package: "swift-openapi-generator"
                )
            ]
        ),
        .executableTarget(
            name: "EventService",
            dependencies: [
                "EventAPI",
                .product(
                    name: "OpenAPIRuntime",
                    package: "swift-openapi-runtime"
                ),
                .product(
                    name: "OpenAPIVapor",
                    package: "swift-openapi-vapor"
                ),
                .product(
                    name: "Vapor",
                    package: "vapor"
                ),
                .product(
                    name: "PostgresNIO",
                    package: "postgres-nio"
                ),
                .product(
                    name: "Logging",
                    package: "swift-log"
                ),
            ]
        ),
    ]
)
```

### 13:38 - Adding logging to the createEvent method

```swift
func createEvent(
    _ input: Operations.createEvent.Input
) async throws -> Operations.createEvent.Output {
    switch input.body {
    case .json(let event):
        do {
            try await self.postgresClient.query(
                """
                INSERT INTO events (name, date, attendee)
                VALUES (\(event.name), \(event.date), \(event.attendee))
                """
            )
            
            return .created(.init())
        } catch let error as PSQLError {
            let logger = Logger(label: "CreateEvent")
            
            if let message = error.serverInfo?[.message] {
                logger.info(
                    "Failed to create event",
                    metadata: ["error.message": "\(message)"]
                )
            }
            
            return .badRequest(.init())
        }
    }
}
```

Inbucation process
run by swift server workgroup
goal to create a robust and stable ecosystem
* sandbox
* incubating
* graduated
each level has different requirements aligned with productionreadyness and uses.

# Wrap up
* develop services in Swift
* powers apple's cloud services
* healthy and growing package ecosystem

[[Meet Swift OpenAPI Generator]]
[[Use Xcode for server-side development]]
[[Beyond the basics of structured concurrency]]



# Resources
* https://www.swift.org/server/
* https://www.swift.org/packages/
* https://www.swift.org/sswg/
