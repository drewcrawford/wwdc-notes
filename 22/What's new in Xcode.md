#xcode 

Dynamic type variants
# Code completion
autocomplete suggests a memberwise initializer.

1.  Default values will not be autocompleted by default
2.  you can opt in by typing the parameter name, e.g. `MyTypeparameterName` and choosing the autocomplete suggestion for the `MyType` initializer.

# Jump to definition
highlights what's different about each option in disambiguation.

# Regex
syntax checking for regex.
# Dimming
Dim grey => xcode is re-evaluating the diagnostic!!!!!!!!!!!!!!!  *FB9073256*
Special shoutout to whoever read my bug report and added this.
Special boo to whatever person at feedback assistant hasn't informed me this was fixed.

# Headers
as you scroll
# build phases
1.  Comple
2.  module
3.  link
4.  link app

in xcode 14, we can eagerly produce modules.
linker 2x faster
overall, 25% faster.

now have "build timeline" view.  Identify unexpectedly long tasks and bottlenecks.

Learn more, [[Demystify parallelization in Xcode builds]]
[[Link fast improve build and launch]]

30% faster testing
[[Author fast and reliable tests for Xcode Cloud]]

4x faster notarizing
IB loads 50% faster
IB  faster device switching

[[Use Xcode to build a multiplatform app]]

# Memory debugger

Memory debugger - see references in an out of objects.  Total weight of objects?

# Package plugins
e.g., codegen.  Compress/optimize, etc.
[[Meet Swift Package plugins]]
[[Create Swift Package plugins]]

# Package localization
[[Building global apps Localization by example]]

# Run destination chooser
Prioritizes recent choices.
filter

# Organizer
Feedback.  Email testers directly.  
Hangs. 
[[Track down hangs with Xcode and on-device detection]]

# Asset catalog
If you don't need pixel hinting, Xcode can create all sizes from 1 image. Inspector => single size.
