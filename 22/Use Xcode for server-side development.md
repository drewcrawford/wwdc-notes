Extending an iOS application itno the cloud.
Many of our apps focus on a single device, usually the iPHone.  AS usage grows, we find ourselves wanting to bring it to additional devices.  Xcode helps us organize and build our applications for these platforms.  We can share code using packages while embracing the unique code for each device.

Enable the client application to extend their functionality into the cloud.  ex, offload tasks that are computationally heavy, etc.  

Server components need to be built using different tools/methodologies.  Creating duplication of effort and integration challenges.  Using swift helps bridge the technology gap.  Let's see what building a server app in swift looks like.

* swift packages
* Defines an executable target that maps the application entrypoint
* We can addd a depdendency on a web framework
```swift
// swift-tools-version: 5.7

import PackageDescription

let package = Package(
    name: "MyServer",
    platforms: [.macOS("12.0")],
    products: [
        .executable(
            name: "MyServer",
            targets: ["MyServer"]),
    ],
    dependencies: [
        .package(url: "https://github.com/vapor/vapor.git", .upToNextMajor(from: "4.0.0")),
    ],
    targets: [
        .executableTarget(
            name: "MyServer",
            dependencies: [
                .product(name: "Vapor", package: "vapor")
            ]),
        .testTarget(
            name: "MyServerTests",
            dependencies: ["MyServer"]),
    ]
)
```

In tihs example, we use the vapor web framework, and open-source community project.  As with other swift-based executables, program's entrypoitn is best modeled with `@main`.

```swift
import Vapor

@main
public struct MyServer {
    public static func main() async throws {
        let webapp = Application()
        webapp.get("greet", use: Self.greet)
        webapp.post("echo", use: Self.echo)
        try webapp.run()
    }

    static func greet(request: Request) async throws -> String {
        return "Hello from Swift Server"
    }

    static func echo(request: Request) async throws -> String {
        if let body = request.body.string {
            return body
        }
        return ""
    }
}
```

Application type used in this exapmle is provided by Vapor.  With basic bootstrapping in place, make our app do something usefl.  ex, greet users making a request to the server.


echoing content of the request body back to the caller.
Here we have our server application in xcode.  Run the server locally on our own machine.  To run locally, we pick the "MyServer" scheme generated in xocde.

Once app has launched, we use xcode console to examine log messages emitted by the server.  Server was started and listening on the localhost address 127.0.0.1:80.  Use this information to test our server.

make a request to the advertised server address with curl.

```bash
curl http://127.0.0.1:8080/greet; echo
curl http://127.0.0.1:8080/echo --data "Hello from WWDC 2022"; echo
```

How to call a server frmo our iOS app?  Example.

```swift
import Foundation

struct MyServerClient {
    let baseURL = URL(string: "http://127.0.0.1:8080")!

    func greet() async throws -> String {
        let url = baseURL.appendingPathComponent("greet")
        let (data, _) = try await URLSession.shared.data(for: URLRequest(url: url))
        guard let responseBody = String(data: data, encoding: .utf8) else {
            throw Errors.invalidResponseEncoding
        }
        return responseBody
    }

    enum Errors: Error {
        case invalidResponseEncoding
    }
}
```

Server resopnse may be more sophisticated than s tring, ex encoded in JSON.



```swift
import SwiftUI

struct ContentView: View {
    @State var serverGreeting = ""
    var body: some View {
        Text(serverGreeting)
            .padding()
            .task {
                do {
                    let myServerClient = MyServerClient()
                    self.serverGreeting = try await myServerClient.greet()
                } catch {
                    self.serverGreeting = String(describing: error)
                }
            }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

Let's deploy our app to the cloud.  In this example, we wil use heroku.  Convenient git push to deploy system for small projects.

Use buildpacks to compile the application remotely.  deploys binary artifacts.  Available for all Swift on server users.  Can test our app with curl.  Let's test the first endpoint.

Let's pause and review the main takeaways
* swift is a general-purpose language
* xcode is a powerful development environment
* choice of cloud providers

1.  Our iOS app has in-memorystorage
2. extract storage and move to the server
3. Allows all users of the app to share the same storage and thus donut menu
4. Our server will expose an HTTP based API.
5. iOS app will use an abstraction for working with those APIS and tie them back to presentation tier.

`Content` is defined by the web framework and helps us encode JSON on the wire.


```swift
import Foundation
import Vapor

@main
struct FoodTruckServerBootstrap {
    public static func main() async throws {
        // initialize the server
        let foodTruckServer = FoodTruckServer()
        // initialize the web framework and configure the http routes
        let webapp = Application()
        webapp.get("donuts", use: foodTruckServer.listDonuts)
        try webapp.run()
    }
}

struct FoodTruckServer {
    private let storage = Storage()

    func listDonuts(request: Request) async -> Response.Donuts {
        let donuts = self.storage.listDonuts()
        return Response.Donuts(donuts: donuts)
    }

    enum Response {
        struct Donuts: Content {
            var donuts: [Model.Donut]
        }
    }
}

struct Storage {
    var donuts = [Model.Donut]()

    func listDonuts() -> [Model.Donut] {
        return self.donuts
    }
}

enum Model {
    struct Donut: Codable {
        var id: Int
        var name: String
        var date: Date
        var dough: Dough
        var glaze: Glaze?
        var topping: Topping?
    }

    struct Dough: Codable {
        var name: String
        var description: String
        var flavors: FlavorProfile
    }

    struct Glaze: Codable {
        var name: String
        var description: String
        var flavors: FlavorProfile
    }

    struct Topping: Codable {
        var name: String
        var description: String
        var flavors: FlavorProfile
    }

    public struct FlavorProfile: Codable {
        var salty: Int?
        var sweet: Int?
        var bitter: Int?
        var sour: Int?
        var savory: Int?
        var spicy: Int?
    }
}
```

One interesting point is that we copied a definition of this model from the foodtruck iOS app, as we need the data model to align.  

Conformance to Encodable.  Required for our server to encode as JSON onver the wire.

Server storage.  Many strategies.
* files for static data
* iCloud for user-centric data
* Database for transactional data
	* there are database drivers for most database technologies.
	* we're going to demonstrate static files, but you can learn more about databases 

```swift
[
  {
      "id": 0,
      "name": "Deep Space",
      "date": "2022-04-20T00:00:00Z",
      "dough": {
          "name": "Space Strawberry",
          "description": "The Space Strawberry plant grows its fruit as ready-to-pick donut dough.",
          "flavors": {
              "sweet": 3,
              "savory": 2
          }
      },
      "glaze": {
          "name": "Delta Quadrant Slice",
          "description": "Locally sourced, wormhole-to-table slice of the delta quadrant of the galaxy. Now with less hydrogen!",
          "flavors": {
              "salty": 1,
              "sour": 3,
              "spicy": 1
          }
      },
      "topping": {
          "name": "Rainbow Sprinkles",
          "description": "Cultivated from the many naturally occurring rainbows on various ocean planets.",
          "flavors": {
              "salty": 2,
              "sweet": 2,
              "sour": 1
          }
      }
  },
  {
      "id": 1,
      "name": "Chocolate II",
      "date": "2022-04-20T00:00:00Z",
      "dough": {
          "name": "Chocolate II",
          "description": "When Harold Chocolate II discovered this substance in 3028, it finally unlocked the ability of interstellar travel.",
          "flavors": {
              "salty": 1,
              "sweet": 3,
              "bitter": 1,
              "sour": -1,
              "savory": 1
          }
      },
      "glaze": {
          "name": "Chocolate II",
          "description": "A thin layer of melted Chocolate II, flash frozen to fit the standard Space Donut shape. Also useful for cleaning starship engines.",
          "flavors": {
              "salty": 1,
              "sweet": 2,
              "bitter": 1,
              "sour": -1,
              "savory": 2
          }
      },
      "topping": {
          "name": "Chocolate II",
          "description": "Particles of Chocolate II moulded into a sprinkle fashion. Do not feed to space whales.",
          "flavors": {
              "salty": 1,
              "sweet": 2,
              "bitter": 1,
              "sour": -1,
              "savory": 2
          }
      }
  },
  {
      "id": 2,
      "name": "Coffee Caramel",
      "date": "2022-04-20T00:00:00Z",
      "dough": {
          "name": "Hardened Coffee",
          "description": "Unlike other donut sellers, our coffee dough is simply a lot of coffee compressed into an ultra dense torus.",
          "flavors": {
              "sweet": -2,
              "bitter": 4,
              "sour": 2,
              "spicy": 1
          }
      },
      "glaze": {
          "name": "Caramel",
          "description": "Some good old fashioned Earth caramel.",
          "flavors": {
              "salty": 2,
              "sweet": 3,
              "sour": -1,
              "savory": 1
          }
      },
      "topping": {
          "name": "Nebula Bits",
          "description": "Scooped up by starships traveling through a sugar nebula.",
          "flavors": {
              "sweet": 4,
              "spicy": 1
          }
      }
  }
]
```

we can make the file accessible with swiftpm's resource support.  Resource file path using swiftpm's dedicated resource accessory.

Load content of files into memory, use JSON decoder to decode content into server application data model.  One interesting change is that `Storage` is an actor.  Now has mutable donuts variable.   Help us deal with shared mutable state in a safe but easy way.

```swift
// swift-tools-version: 5.7

import PackageDescription

let package = Package(
    name: "FoodTruckServer",
    platforms: [.macOS("12.0")],
    products: [
        .executable(
            name: "FoodTruckServer",
            targets: ["FoodTruckServer"]),
    ],
    dependencies: [
        .package(url: "https://github.com/vapor/vapor.git", .upToNextMajor(from: "4.0.0")),
    ],
    targets: [
        .executableTarget(
            name: "FoodTruckServer",
            dependencies: [
                .product(name: "Vapor", package: "vapor")
            ],
            resources: [
                .copy("menu.json")
            ]
        ),
        .testTarget(
            name: "FoodTruckServerTests",
            dependencies: ["FoodTruckServer"]),
    ]
)
```

Wire up the bootstrap to the executables entrypoint.  Since storage is now an actor, we access in an async context.

Server abstraction that will help us encapsulate server APIs.  Use URL session, JSON decoder, etc.



```swift
import Foundation
import Vapor

@main
struct FoodTruckServerBootstrap {
    public static func main() async throws {
        // initialize the server
        let foodTruckServer = FoodTruckServer()
        try await foodTruckServer.bootstrap()
        // initialize the web framework and configure the http routes
        let webapp = Application()
        webapp.get("donuts", use: foodTruckServer.listDonuts)
        try webapp.run()
    }
}

struct FoodTruckServer {
    private let storage = Storage()

    func bootstrap() async throws {
        try await self.storage.load()
    }

    func listDonuts(request: Request) async -> Response.Donuts {
        let donuts = await self.storage.listDonuts()
        return Response.Donuts(donuts: donuts)
    }

    enum Response {
        struct Donuts: Content {
            var donuts: [Model.Donut]
        }
    }
}

actor Storage {
    let jsonDecoder: JSONDecoder
    var donuts = [Model.Donut]()

    init() {
        self.jsonDecoder = JSONDecoder()
        self.jsonDecoder.dateDecodingStrategy = .iso8601
    }

    func load() throws {
        guard let path = Bundle.module.path(forResource: "menu", ofType: "json") else {
            throw Errors.menuFileNotFound
        }
        guard let data = FileManager.default.contents(atPath: path) else {
            throw Errors.failedLoadingMenu
        }

        self.donuts = try self.jsonDecoder.decode([Model.Donut].self, from: data)
    }

    func listDonuts() -> [Model.Donut] {
        return self.donuts
    }

    enum Errors: Error {
        case menuFileNotFound
        case failedLoadingMenu
    }
}

enum Model {
    struct Donut: Codable {
        var id: Int
        var name: String
        var date: Date
        var dough: Dough
        var glaze: Glaze?
        var topping: Topping?
    }

    struct Dough: Codable {
        var name: String
        var description: String
        var flavors: FlavorProfile
    }

    struct Glaze: Codable {
        var name: String
        var description: String
        var flavors: FlavorProfile
    }

    struct Topping: Codable {
        var name: String
        var description: String
        var flavors: FlavorProfile
    }

    public struct FlavorProfile: Codable {
        var salty: Int?
        var sweet: Int?
        var bitter: Int?
        var sour: Int?
        var savory: Int?
        var spicy: Int?
    }
}
```

With the application running, let's see that we can access the APIs.

```bash
curl http://127.0.0.1:8080/donuts | jq .
curl http://127.0.0.1:8080/donuts | jq '.donuts[] .name'
```

Switch to xcode, pick food truck scheme.  Simulator.  Run it.  Cross-reference with server. 

Can share the model across iOS and server applications.
1.  Create another package for the library, and add it to the xcode workspace
2. move data model code to the shared package, etc.
3. refactor our client code to use the shared model
4. do the same to the server code

# Next steps
* menu editor
	* will require we move our storage to database
* buying and ordering
* dynamic pricing

# WRap up
* swift is a general-purpose language
* Code sharing between server and client
* Codable for safe serialization
* URLSession to interact with server
* Xcode is a pwoerfuld evelopment environment

[[Use Swift on AWS Lambda with Xcode]]
[[Protect mutable state with Swift actors]]
[[Meet distributed actors in Swift]]




* https://developer.apple.com/forums/tags/wwdc2022-110360
* https://developer.apple.com/forums/create/question?&tag1=266&tag2=525030
* https://www.swift.org/server/
