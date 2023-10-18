#location 
Discover how Core Location Monitor can help you better understand location and beacon events in your app. Learn how to use Core Location Conditions to describe and track the state of events in your app, and find out how you can better respond to transitions in your apps through Swift semantics and improved reliability.

Enables you to write simple yet powerful... with just a few lines of code in swift.  You just create a monitor, add conditions, and await events.

# Monitor overview

CLMonitor and friends
Each CLMonitor instance acts as a gateway.  Relieeves you from overhead of threading, etc.

To create a monitor, you call `await CLMonitor("Greeter")`.  Automatically reused, etc.

Only one instance with a given name can be open at any moment.

The entity being monitored is called a Condition.  

Identifier.  

# Supported conditions
`CircularGeographicCondition`.  Defined by a center anda  radius.  The center, defines the geographic center.

Radius -> area.  Anywhere outside the area, we report as unsatisfied.  Similar to... regions?

* CircularGeographicCondition
* BeconIdentityCondition
* Any side
* specific site
* specific section

Beacon
* UUID
* major string
* minor string

You can monitor a particular site with a more specific condition.

When you add a condition, its initial state will be unknown.

You might be aware of the current state.  In those cases, you can pass in a state anyway with `assuming: .unsatisfied`.  


# Inspecting records
CL creates record and adds condition.  There's also a record event.

Event contains
* state (satisfied, unsatisfied, unknown)
* date
* condition? ("refinement")  This has something to do with the fact that you can request any combo of uuid/major/minor.  Not sure why.

We keep track of all records from your app?


# Handling events
Can implement with a simple try/await loop.


# Best practices

I have some advice on how to use it best.

* Keep monitor instances unique (for a given name)
* (always have a) task to await events
* If terminated, when launched:
	* reinint monitor
	* await events again
* Use CLMonitor only from your app
* Leverage CLMonitor's tracked state
* Reserve parallel state for special cases

# Monitor your away
* Try CLMonitor
* Provide feedback
* Try our sample
[[Discover streamlined location updates]]

# Resources
* https://developer.apple.com/documentation/corelocation
* https://developer.apple.com/documentation/corelocation/monitoring_location_changes_with_core_location
* https://developer.apple.com/documentation/corelocation/adopting_live_updates_in_core_location
