#xcode 

Strategies and techniques for making the most of your build configuration.

# Multi-platform frameworks
* Simplified target management
* One set of build phases
* One set of build settings

Build settings, sources, tough to keep in sync.

Convert one framework to multi-platform.

Supported Platforms => Any platform.  Allow multiplatform builds => YES.

Allows the build system to build multiple times for each target.

Exclude files?  Add a platform filter.

Build Phases=>Compile sources=>Filters

Delete the other two variants of the framework as these are no longer needed.  Because we have only 1 framework target, we have to configure all of our apps to embed/link that framework.

* Converted an existing framework to multi-platform
* Configured platform filters to customize the build for each platform
* Configured our apps to link/embed the new multiplatform framework.



# Project model and configuration
Improve performance/correctness of your builds.

Scheme settings=>build.

We recommend "Dependency order" for build order.  In contrast, "Manual Order" will slow down your build.

Also consider "Find Implicit Dependencies" to automatically ad dependencies between targets based on linker flags, build settings, etc.

Important when targets are in different projects.

If you're using manual dependency order due to this, enabling "find implicit dependencies" is often a better solution.

Build Rules.  Need to tell the build system the output file.
`$(DERIVED_FILE_DIR)/$(INPUT_FILE_BASE).compiledrecipe`

Best practice to use $DERIVED_FILE_DIR.  Avoid generating otuput files under sourceroot, which leads to conflicts with multiple simultaneous builds.

Remember that rules run once for each input they process.  

Don't forget to quote variables to ensure spaces etc. ar ehandled.

By default, rules run once for each *architecture* as well.  e..g 1once for arm64, once for x64.  4 inputs * 2 architectures = 8x.

Useful when output is architecture-dependent.  However, my rule is independent of CPU architecture, so I uncheck "Run once per archicture"

All `.recipes` must be a "compile sources"

A build *rule* isn't appropriate for this, because we need to process all inputs to combine them into 1.  So can't break up into parallel units.  Makes sense to keep this linklike workload in a script phase.

But since therea re no dependencies, might be wrong order or slow down build.  Important to add input/output dependencies to ensure the work performed by script phases is done in the correct order.

xcfilelist can manage inputs in an external file.  File=>New file => Build phase file list.

You can even write comments by beginning a line with `#`.

Reference this `xcfilelist` from script phase.

Specify an output dependency.

`${SCRIPT_INPUT_FILE_LIST_COUNT}`
`SCRIPT_INPUT_FILE_LIST_2` etc.

`SCRIPT_OUTPUT_FILE_0` etc.

```
// These environment variables are available in script phases:

SCRIPT_INPUT_FILE_COUNT // This specifies the number of paths from the Input Files table.
SCRIPT_INPUT_FILE_n // This specifies the absolute path of the nth file from the Input Files table, with build settings expanded.

SCRIPT_INPUT_FILE_LIST_COUNT // This specifies the number of input file lists.
SCRIPT_INPUT_FILE_LIST_n // This specifies the absolute path of the nth "resolved" input file list with contained paths made absolute, build settings expanded, and comments removed.

SCRIPT_OUTPUT_FILE_COUNT // This specifies the number of paths from the Output Files table.
SCRIPT_OUTPUT_FILE_n // This specifies the absolute path of the nth file from the Output Files table, with build settings expanded.

SCRIPT_OUTPUT_FILE_LIST_COUNT // This specifies the number of output file lists.
SCRIPT_OUTPUT_FILE_LIST_n // This specifies the absolute path of the nth "resolved" output file list with contained paths made absolute, build settings expanded, and comments removed.

* n in the above examples refers to a 0-based index.
```

Each build in a multiplatform build might produce the same item.  Not allowed.

Simple solution is to change the output path so it's unique each time the target is built.  e.g., `$DERIVED_FILE_DIR`.

However, if the work would be identical within the context of each target, that would cause the same work to be done twice.  In that case, maybe move script phase to an aggregate target which the shared target depends on.


# Build settings deep dive

What is a build setting?  Property to your xcode targets to configure build

2 mechanisms
* Build settings editor
* configuration settings file (`.xcconfig`)

## Editor

Levels.  Right to left.

1.  Default value (SDK)
2.  configuration settings file
3.  project settings (editor)
4.  target settings (config file)
5.  target settings (editor)
6.  Resolved

Bold means it has an explicit value

## configuration settings files

1.  Better source control management
2.  Sharing settings across targets or configurations
3.  Advanced composition of build settings
4.  Include additional configuration based on dev/test

## Definition
`MY_BUILD_SETTING_NAME = "a build setting value"`

Conditional settings, with square brackets
```
MY_BUILD_SETTING_NAME = "A build setting value"
MY_BUILD_SETTING_NAME[config=Debug] = -debug_flag
MY_BUILD_SETTING_NAME[arch=arm64] = -arm64_only
MY_BUILD_SETTING_NAME[sdk=iphone*] = -ios_only
```
Wildcards can be used as shown.

Comments using `//`.  

`$()` evaluates another setting.

`$(inherited) -more_flag`

Can also use build setting name ot append to existing settings.

Can also compose settings together.

## composition

```
IS_BUILD_SETTING_ENABLED = NO
MY_BUILD_SETTING_NO = -use_this_one
MY_BUILD_SETTING_YES = -use_this_instead
MY_BUILD_SETTING = $(MY_BUILD_SETTING_$(IS_BUILD_SETTING_ENABLED))
```

## Evaluation operators
* String operators
* path operators
* replacement operators


| Operator                          | Value                       | Transformation                    |
|-----------------------------------|-----------------------------|-----------------------------------|
| `$(MY_SETTING:quote)`             | character will be "escaped" | characters\ will\ be\ \"escaped\" |
| `$(MY_SETTING:lower)`             | All Caps Will Be Lowered    | all caps will be lowered          |
| `$(MY_SETTING:upper)`             | make it RAIN                | MAKE IT RAIN                      |
| `$(MY_SETTING:identifier)`        | valid c99 identifier        | valid_c99_identifier              |
| `$(MY_SETTING:c99extidentifier)`  | valid c99 ext ïdentifięr    | valid_c99_ext_ïdentifięr          |
| `$(MY_SETTING:rfc1034identifier)` | a valid RFC1034 identifier  | A-valid-RFC-1034-identifier       |

### paths

| Operator                    | value                                      | transformation                     |
|-----------------------------|--------------------------------------------|------------------------------------|
| `$(MY_PATH:dir)`            | /projects/goodies/src/winner.swift         | /projects/goodies/src              |
| `$(MY_PATH:file)`           | /projects/goodies/src/winner.swift         | winner.swift                       |
| `$(MY_PATH:base)`           | /projects/goodies/src/winner.swift         | winner                             |
| `$(MY_PATH:suffix)`         | /projects/goodies/src/winner.swift         | swift                              |
| `$(MY_PATH:standardizepath)` | /projects/goodies/src/oops/../winner.swift | /projects/goodies/src/winner.swift |


### replacement
Can use `dir=/tmp`, `file=/better.swift` etc. to replace the part of the path

Also `default=YES` to override the default value, not a value that is set.

## Include other settings
`#include "path/to/file.xcconfig"`

Requires to exist on disk.

`#include? "path/to/file.xcconfig"`
Will *not* fail if file DNE

Path is relative to xcode project file.

# Practical example
On dev, agressively warn for typecheck.

On CI, machine is slower?

1.  Debug
	1.  `OTHER_SWIFT_FLAGS=-DFRUTA_DEBUG`
2.  common
	1.  Optionally includes ci.xcconfig
	2.  `OTHER_SWIFT_FLAGS=$(inherited) -warn-long-expression-type-checking=$(MAX_EXPRESSION_TIME:default=200)`
3.  CI

Apply any config files between project or target levels from any defined build configuration.

`MAX_EXPRESSION_TIME` has a defualt of 200
ci.xcconfig is optionally included (only exists on CI)
ci.xcconfig overrides `MAX_EXPRESSION_TIME` to 600

# Wrap up
* Multi-platform frameworks
* Project model and configuration
* Build settings deep dive





