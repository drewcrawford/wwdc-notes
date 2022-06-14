#xcodecloud 

# Xcode Cloud overview
tools to run apps, etc.
[[Meet Xcode Cloud]]

# Review an existing Workflow
usage - total time all actions.  See distribution

duration vs usage.  Duration of build is the longest running action (because actions run in parallle).  For usage, each one in sequence gives us the total usage.
# xcode Cloud usage dashboard

# Best practices
* Avoid unintended builds

[[Explore Xcode Cloud Workflows]]

Don't build unless we have changes.  Vs scheduled build.
Can use custom conditions to exclude files.

Select reasonable test destinations.  Each destination runs in parallel.  Ensure concise set of simulator destinations.

Xcode cloud has an alias for recommended destinations.  Simulator cross-section of screen sizes.

Skip builds.  `[ci skip]`.  Now xcode cloud knows to ignore this event.  Use exact format shown here.

Optimize custom scripts and tests.  Retry API requests etc.

[[Customize your advanced Xcode Cloud workflows]]

Spend more time writing reliable tests.

# Revisit optimized build

[[Deep Dive into Xcode Cloud for Teams]]

