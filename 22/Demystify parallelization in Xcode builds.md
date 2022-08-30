Deep dive into xcode's build process.

# Core concepts
[[Behind the scenes of the Xcode build process - 18]]

How does the build system decide an order to execute tasks?

Swift c omipler captures the program's intent and translates it into binary.  Can fail, which will cancel the build, if it succeeds it creates .o for each input.

Those are combined by the linker.  Two tasks have a dependency.  Object files are produced by the compiler, then consumed by the linker.  This creates a dependency.

Needs to make sure a task that produces an input finishes before the other task can start.

A => produces some or all of B's inputs.

There are many cases besides compiling and linking.  Those tasks have defined dependencies based on inputs and outputs.

Tasks that get unblocked are downstream, and tasks that block are upstream.

frameworks, etc.  

When executing the build graph, different tasks takea  different amount of time.  Level of complexity necessary to complete the work.  Compling many files takes more time than header files.

Build system starts by running tasks that don't have dependencies.  Then downstream tasks, etc.

Build system is able to skip tasks for inputs that haven't changed.  If a task needs to rerun due to changed inputs, downstream tasks have to rerun too if it's also changed.  Incremental builds.

Critical path.  Common pattern throughout this talk is to shorten this path.  Ashorter critical path does not necessarily result in a smaller buildtime, but it ensures build scales to the hardware.  A build cannot complete faster if hardware would allow it.

New visualization in xcode 14.  Helps understand build's performance after finished.  Visualize based on parallelization rather than hierarchy
Number of rows represents parallelism
Horizontal length of tasks represents duration
Empty space shows tasks blocking the build
colors differentiate targets
Contains only executed tasks

## Demo
To get an overview of built targets, let's check out the scheme editor
Build tab contains a list of all targets.  Explicitly added to scheme or implicitly.  In this case, I'm using a swift package with an automatically generated scheme with swift package.

Upper right corner, open assistant.  This seems to be in the build log.
Visualizes the same data in the recent build log.  Selecting in one selects it in the other.  See execution in context.

Use a pinch gesture to zoom.  Since the build log visualizes based on hierarchical structure, see what was compiled.  Also enables to view the whole command.

Holding down option when selecting an area in the build timeline adjusts the viewport to this timeframe.  Verify the linking of argument parser.

Holding option by scrolling up, allows me to zoom more quickly.  Number of tasks that run in parallel.  Tasks waiting for unproduced inputs.

XCode comes with many improvements to shorten the critical path.
# Build phases
Build phases describes work that needs to be done.  Many build phases describe tasks with inputs/outputs, creating dependencies.  For example, source files are compiled before linked.

Instead of running build phases in a linear order, build system will consider inputs/outputs to see if they can run in parllel.  e.g., resource copying in parallel with compiling.

Consider run script phases.  These must be manually configured in the editor.  Tehrefore, build system will run them one at a time to avoid data races.  You can opt in via `FUSE_BUILD_SCRIPT_PHASES`.  This indicates the build system can run them in parallel.

Running them in parallel, the build system has to rely on inputs and outputs.  An underspecified input or output dependencies can lead to data races!

`ENABLE_USER_SCRIPT_SANDBOXING`.  Will deny access to unallowed resources.  In this example, neither input or output are declared as a dependency, so sandbox will block script.  Sandbox ensures script is not mistakenly accessing any other values.

How sandboxing prevents data races etc.

## ex
Without sandboxing a misconfiguration can be unnoticed.  It means xcode will fail to infer the dependency and schedule to run in parallel.  Second task can run first.

Or, xcode may pick up outdated file.  executing in sandbox prevents this issue.

To enable "Run Build Script Phases in Parallel" or "User Script Sandboxing" in project build settings.

## summary
* faster and more robust incremental builds
* Blocks access in `PROJECT_DIR` and intermediate build files for undeclared inputs/outputs
* don't ocnsider this a security feature, that's all we block
* helps diagnose misisng dependency information for script phases
* Parallel script execution with FUSE_BUILD_SCRIPT_PHASES

# Cross-target builds

Task graphs can increase in size and complexity.  Whiel xcode system breaks down into a sea of tasks taht corresonds to all targets.

One special task is compilation.  Building a sourcecode itno a binary product is a complex operation which has many subtassks for planning, compilation, and linking.  Coordination is delegated to the Swift Driver.  Has special knowledge.  

Binary module file capturing public interface isa  build product taht is required for downstream targets to begin compilation.

## ex
In release/optimized,
1.  One compiler task contianing all sourcefiles
2. produces target swift module

debug/incremental
1.  Small subtasks that run in parallel
2. merge module
If the number of files is high, may be more than one compilation in a job.

Being able to parallelize is crucial for faster/smaller incremental builds.  Make sure debug builds are using "Incremental" build setting.

Before xcode 14, because of the boundary between xcode and swift driver, orchestration of target build phases and compilation tasks spawned by each target's instance of the driver happened independently of each other.

With xcode 14, thanks to new implementation of swift driver, build system and compiler are integrated.  Central planning allows xcode to make fine-grained scheduling decisions.  

What was previously a collection of islands of subtasks are now fully in the build system.

On m1, it assigns available tasks to 1 core.  On higher core, we perform more work.  Also more likely to have idle cores which are available to perform more work, but all outstanding tasks are awaiting their inputs.  Integrated build system reduces the idle time.

new in xcode 14, construction of a target's module is done in a special task directly from all sourcefiles.  So a target's dependencies can begin compialtion as soon as `emit-module` completes.  Unblocking downstream compilation cuts down on available work.

Extenidng to the rest of our project shows that although we're performing a similar amounto f overall work, we can use resources more efficiently.

## Eager linking
Linker task for each target are on critical path.  Because B links A, B must wait for link output from A.

However, with eager linking, target B's link task can depend on `emit-module` task instead.  So B can begin linking earlier in the build.

Instead of depending on a linked product, it depends on a dynamic library stub built by `emit-module`.

Enable using "Eager Linking" setting.  Applies to all **pure swift targets** that are **dynamically linked by their dependents.**

# Takeaways
* build phases can be executed in parallel
* sandboxing makes incremental buidls faster and more reliable
* xcode and swift are more integrated than ever
* modularized builds scale with available hardware resources
* The build timeliine is a powerufl new tool for ganing insight into build performance
* SwiftDriver github.com/apple/swift-driver

# next steps
[[What's new in Xcode]]
[[Link fast improve build and launch times]]


* https://github.com/apple/swift-driver