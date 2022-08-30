Getting started with package plugins.  xcode 14 extends to development workflow, letting you use plugins to do things like generating sourcecode.

# What is a package plugin?
Swift script that can perform pactions on a package or xcode project.  Uses xcode API.

Can provide library and executables.

Highly specialized plugin can be private to teh package that provides it.  General-purpose plugin can be made available to other packages.  

Unlike a library, a dependency on a plugin does nto bring runtime content into your app.  Lets you access development tools for your machine.

2 kinds of plugins.
1.  Comamnd plugin
2. build tool plugin

Things you do with arbitrary scripts todya.  Can ask for permission to modify files in a package.  

Build tool plugins extend teh build system's dependency graph.  Useful for generating sourcecode or resources part of a build.  Applied to each target that needs them.

## Demo
Splitting out this package so that others can use it. 

Add package dependency in my manifest, just like I would for a library.  Xcode fetches the remote package and it appears in the dependencies.  

I have access to any plugin commands that the package provides.  Right click my package and see special actions.  

Xcode let sm e choose which target to pass to the plugin.  I can invoke on the whole package.  Pass custom arguments.  Click run, because the plugin will modify the filesystem, xcode warns me.  "Show command" to see implementation.


# How do plugins work?
Package plguins are swift scripts that are compield and run when they are needed.  Each plugin runs as a separate process.  Plugins have access to a distilled representation of the input package.

many plugins call command line tools aspart of doing their work.  Can create file sand directories, and perform other actiosn with foundation.  Runs in a sandbox that prevents network access and only allows writing to a few places int he FS.  Command plugins can ask for permission to also modify files in the source directory.

Plugin can send results back to xcode.  Emit warnings and errors, and build tool plugins can deifne tooling locations.

Use API From the `PackagePlugin` module provided by xcode.  Allows plugin to access the input package.

`@main` entry point.  Should be a class or struct that ocnforms to a protocol that matches the type of plugin.  

[[Create Swift Package plugins]]


# Command plugins
*Command plguins* implement development-time actions
* run interactively, not during a build
* Can ask for permission to modify package sources
* Usually depend on other tools to do the actual work

SwiftFormat.  Xcode will build any required tools from source.  Plugin can be provided by a different package than the tool it relies on.

```swift
import PackagePlugin

@main
struct MyPlugin: ... {

    // Entry points specific to plugin capability. These entry points are invoked
    // when the plugin is applied to a package.

}

#if canImport(XcodeProjectPlugin)
import XcodeProjectPlugin

extension MyPlugin: ... {

    // Entry points specific to plugin capability. These entry points are invoked
    // when the plugin is applied to an Xcdeo project.

}
#endif
```

```swift
import PackagePlugin

@main
struct MyPlugin: CommandPlugin {

    /// This entry point is called when operating on a Swift package.
    func performCommand(context: PluginContext, arguments: [String]) throws {
        debugPrint(context)
    }
}

#if canImport(XcodeProjectPlugin)
import XcodeProjectPlugin

extension MyPlugin: XcodeCommandPlugin {

    /// This entry point is called when operating on an Xcode project.
    func performCommand(context: XcodePluginContext, arguments: [String]) throws {
        debugPrint(context)
    }
}
#endif
```

swiftpm 5.6 has a new subcommand for plugins.  



```bash
swift package --allow-writing-=to-package-directory plugin-name
```



# Build tool plugins
Unlik ea command plugin, build tool plugins provide commands for the build system
* can invoke executables provided as binaries or built from source
* supports build commands and prebuild commands
* output files are stored with other build artifacts, not among package sources
* commands run in a sandbox that prevents changes to the package

```swift
import PackagePlugin

@main
struct MyPlugin: BuildToolPlugin {

    /// This entry point is called when operating on a Swift package.
    func createBuildCommands(context: PluginContext, target: Target) throws -> [Command]
        debugPrint(context)
        return []
    }
}

#if canImport(XcodeProjectPlugin)
import XcodeProjectPlugin

extension MyPlugin: XcodeBuildToolPlugin {

    /// This entry point is called when operating on an Xcode project.
    func createBuildCommands(context: XcodePluginContext, target: XcodeTarget) throws -> [Command]
        debugPrint(context)
        return []
    }
}
#endif
```

Buidl commands run as part of the build
* specify input paths and output paths
* run when outputs are missing, or inputs change
prebuild commands run before the build starts
* can generate outputs whose names can't be known beforehand
* run before every build
	* do as little work as possible when there are no changes

In swiftpm 5.6 and later, theres a `plugins` paramater on each target.  Specifies any plugin needed by a target.  Plugins can be in same package or another package.

In this case, I ahve a custom CLI tool to generate swift code based on data files.  

To get access to the plugin, I go to the manifest and add a dependency on the source generator plguin I want to use.

Similarities and difference between command plugins and build tool plugins

# Wrap up
powerful new way to automate your development workflow
* extend build system commands
* automate development tasks

[[Create Swift Package plugins]]
[[Adopting Swift packages in Xcode - 19]]




