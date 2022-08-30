#swiftpm  #xcode 

We introduced siwft packages in xcode 11.

# What is a package plugin?
swift code that uses package plugin API.  Extend the functionality of xcode or package manager, etc.

how do they work?
1.  Xcode compiles and runs the plugin
2. uses information about execs and input files to construct commands
3. which it tells xcode about

what can it do
* custom build tasks
* custom commands

[[Meet Swift Package Plugins]]
[[Creating Swift packages - 19]]

# Custom commands
`Plugin` folder.
Changes to package manifest.  Bump tools version to 5.6.  Plugins only available since then.
```swift
// MARK: Plugins

        .plugin(
            name: "GenerateContributors",
            capability: .command(
                intent: .custom(verb: "regenerate-contributors-list",
                                description: "Generates the CONTRIBUTORS.txt file based on Git logs"),
                permissions: [
                    .writeToPackageDirectory(reason: "This command write the new CONTRIBUTORS.txt to the source root.")
                ]
            )),
```

corresponds to `Plugin` folder.  Name (folder name).  Custom command.  Intent can define a verb for CLI as well as a description of what the plugin does.
Declare permissions that the plugin requires.  

```swift
import PackagePlugin
import Foundation

@main
struct GenerateContributors: CommandPlugin {

    func performCommand(
        context: PluginContext,
        arguments: [String]
    ) async throws {
        let process = Process()
        process.executableURL = URL(fileURLWithPath: "/usr/bin/git")
        process.arguments = ["log", "--pretty=format:- %an <%ae>%n"]

        let outputPipe = Pipe()
        process.standardOutput = outputPipe
        try process.run()
        process.waitUntilExit()

        let outputData = outputPipe.fileHandleForReading.readDataToEndOfFile()
        let output = String(decoding: outputData, as: UTF8.self)

        let contributors = Set(output.components(separatedBy: CharacterSet.newlines)).sorted().filter { !$0.isEmpty }
        try contributors.joined(separator: "\n").write(toFile: "CONTRIBUTORS.txt", atomically: true, encoding: .utf8)
    }
}
```

To shell out to git, we import foundation.

Create a pipe to capture process output.  Wait until it exits, etc.

In the dialog, we can select the packages or targets that should be the input.  But since our plugin doesn't react to this, we can click run. 

Asked for permissions.  

> Since we just wrote the plugin ourselves, we can go ahead and run it.

# plugins in detail
## Permission model
* sandboxing
* plugin work directory
* requesting permissions

## build tool plugins
* executables
* inputs
* outputs

similar to run script phases.

### types
* in-build command ("defined set of outputs")
	* reruns when outputs are out of date
* pre-build command ("every build")
	* be careful about doing expensive work, or come up with a custom strategy for caching results.

# In-build commands


```swift
platforms: [
            .macOS("10.15"),
            .iOS("12.0"),
            .tvOS("12.0"),
            .watchOS("6.0"),
        ],
```

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Image("Xcode", bundle: .module)
            .resizable()
            .frame(width: 200.0, height: 200.0)
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

```swift
.executableTarget(name: "AssetConstantsExec"),
```


Get run at the start of the build process.  Based on that, executables get scheduled as part of build execution.

We'd like to have a comiple-time ocnstant for each image.  Instead of needing to remember the correct strings, we get them autocompleted as swift symbols.  Loop over directory catalog, parse its metadata, etc.

Deal with arguments

```swift
import Foundation

let arguments = ProcessInfo().arguments
if arguments.count &lt; 3 {
    print("missing arguments")
}
let (input, output) = (arguments[1], arguments[2])

struct Contents: Decodable {
    let images: [Image]
}

struct Image: Decodable {
    let filename: String?
}

var generatedCode = """
    import Foundation
    import SwiftUI
    
    """

try FileManager.default.contentsOfDirectory(atPath: input).forEach { dirent in
    guard dirent.hasSuffix("imageset") else {
        return
    }

    let contentsJsonURL = URL(fileURLWithPath: "\(input)/\(dirent)/Contents.json")
    let jsonData = try Data(contentsOf: contentsJsonURL)
    let asset🐱alogContents = try JSONDecoder().decode(Contents.self, from: jsonData)
    let hasImage = asset🐱alogContents.images.filter { $0.filename != nil }.isEmpty == false

    if hasImage {
        let basename = contentsJsonURL.deletingLastPathComponent().deletingPathExtension().lastPathComponent
        generatedCode.append("public let \(basename) = Image(\"\(basename)\", bundle: .module)\n")
    }
}

try generatedCode.write(to: URL(fileURLWithPath: output), atomically: true, encoding: .utf8)
```

Take advantage of swift's built-in json decoding.  List of images and the filename, which are optional.  Might not be an image for each pixel density.

```swift
.plugin(name: "AssetConstants", capability: .buildTool(), dependencies: ["AssetConstantsExec"]),
```
entrypoint called once per target that uses the given plugin.
This cares particularlly about source module targets.  Rather than e.g. binary targets.
expect a string for the display name as well as suitable input and output path.
Lookup executable using plugin API, and then put out build command together.

Now we can take a look at build log, etc.
Plugin is compiled and run at the start of the build, adds generated commadns to the build graph.

```swift
guard let target = target as? SourceModuleTarget else {
                    return []
                }

        return try target.sourceFiles(withSuffix: "xcassets").map { asset🐱alog in
                    let base = asset🐱alog.path.stem
                    let input = asset🐱alog.path
                    let output = context.pluginWorkDirectory.appending(["\(base).swift"])

                    return .buildCommand(displayName: "Generating constants for \(base)",
                                         executable: try context.tool(named: "AssetConstantsExec").path,
                                         arguments: [input.string, output.string],
                                         inputFiles: [input],
                                         outputFiles: [output])
                }
```

# Pre-build commands
Can share them in a straightforward way similar to libraries.  For the next demo, let's automate prebuild processing with NSLocalizedString
* extract to Base.lproj
* genstrings

[[Swift packages Resources and Localization - 20]]
[[Localize your SwiftUI app]]

1.  compute localization dir
2. compute input
3. construct build command for genstrings

We don't declare a well-defined set of outputs, so these commands run on every build.

add target to manifest
```swift
.plugin(name: "GenstringsPlugin", capability: .buildTool()),
```
we also add a plugin "product".  Similar to library products, this makes a plugin available to clients of a package rather than privately.

```swift
.plugin(name: "GenstringsPlugin", targets: ["GenstringsPlugin"]),
```

```swift
guard let target = target as? SourceModuleTarget else {
                    return []
                }

        let resourcesDirectoryPath = context.pluginWorkDirectory
                    .appending(subpath: target.name)
                    .appending(subpath: "Resources")
                let localizationDirectoryPath = resourcesDirectoryPath
                    .appending(subpath: "Base.lproj")

                try FileManager.default.createDirectory(atPath: localizationDirectoryPath.string, withIntermediateDirectories: true)

        let swiftSourceFiles = target.sourceFiles(withSuffix: ".swift")
                let inputFiles = swiftSourceFiles.map(\.path)

        return [
                    .prebuildCommand(
                        displayName: "Generating localized strings from source files",
                        executable: .init("/usr/bin/xcrun"),
                        arguments: [
                            "genstrings",
                            "-SwiftUI",
                            "-o", localizationDirectoryPath
                        ] + inputFiles,
                        outputFilesDirectory: localizationDirectoryPath
                    )
                ]
```
test the plugin out in a separate example package.

Let's create a new package from template.  Add an API that provides a localized string to a package.  Add use of that to the generated test.
```swift
import Foundation

public func GetLocalizedString() -&gt; String {
    return NSLocalizedString("World", comment: "A comment about the localizable string")
}
```
Test works as our API returns 'world'.  Add a path-based dependency.  



```swift
.package(path: "../GenstringsPlugin"),
```
Add use of the plugin to our library target.

```swift
plugins: [ .plugin(name: "GenstringsPlugin", package: "GenstringsPlugin"), ]
```

If we look at the build log, our plugin gets executed at the start of the build.  Generated files get added to target.  Resource bundle build, and a resource accessor being generated, just as if the reosurce was part of our target from the beginning.

# Wrap up
* Plugins
* custom commands
* build tools






* https://developer.apple.com/forums/tags/wwdc2022-110401
* https://developer.apple.com/forums/create/question?&tag1=236&tag2=558030&tag3=235
