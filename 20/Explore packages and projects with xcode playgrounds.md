#playgrounds
#swiftpm

Evidently double-clicking `Package.swift` opens xcode package now

Playgrounds are stored in the `Playgrounds` folder of a package.  Good choice for documentation, etc.

Also available in projects that contain the package dependency.

## build active scehemes

When "Build Active Schemes" in the righthand panel, the playground could import various frameworks.

Enabled by default for new playgrounds.

Ensure the module you want to import is either part of the scheme, or built by a dependency in that scheme.


## build logs

Now have build logs for supporting the playground.

## Build system
Note that all resources supported by the build system are now supported in playgrounds.  e.g. coreml, etc.