# Impact of memory footprint
Improves user experience.  Faster application activations.  Prevent system from terminating.  Increases chance of application remaining in memory.

Responsive experience. By being strategic about what your app loaded, your app can avoid the cost of waiting to reclaim memory.

Wider-range of features.  Wider device compatibility.  By monitoring your app's memory footprint, it will activate faster, etc.

* Dirty
	* Written by an app
	* All heap allocations
	* Decoded image buffers
	* Frameworks
* Compressed
	* Dirty pages that haven't recently been accessed.  
	* No swap on iOS.
* Clean
	* Memory mapped files
	* Frameworks

Memory footprint = dirty memory + compressed memory.  Clean memory doesn't count.

[[iOS memory deep dive - 18]]


# Tools for profiling memory
XCTest helps monitor footprint in unit/ui test
metrickit and xcode organizer in production

## Performance XCTests #xctest 
* Memory utilization
* CPU usage
* disk writes
* hitch rate
* etc.

```swift
// Monitor memory performance with XCTests

func testSaveMeal() {
    let app = XCUIApplication()
        
    let options = XCTMeasureOptions()
    options.invocationOptions = [.manuallyStart]
        
    measure(metrics: [XCTMemoryMetric(application: app)],
            options: options) {

        app.launch()

        startMeasuring()

        app.cells.firstMatch.buttons["Save meal"].firstMatch.tap()
            
             let savedButton = app.cells.firstMatch.buttons["Saved"].firstMatch
        XCTAssertTrue(savedButton.waitForExistence(timeout: 30))
    }
}
```


Iteration average.  Decide if i want to set an average to compare future tests.  Regression is deviated from the baseline.

## Diagnostic collection in XCTests
* Ktrace files
* Memory graphs

## Ktrace files

[[getting started with instruments]]
[[Understand and eliminate hangs from your app]]

Open/analyze in instruments.

## Memory graph
Used with xcode's visual debugger and a variety of cli tools.  Essentially a snapshot of processes' address space.

Allows you to inspect individual blocks.

Use `xcodebuild test -enablePerformanceTestsDiagnostics YES`.
Enables Ktrace collection for non-memory metrics, and memory for memory metrics.

Output also calls out that the test failed due to a regression.  New average is 12% worse than baseline.

Find new path to XCResult bundle.  Memory measurements at the top next to test name.  Expand test logs and find our attached memgraphs.

2 memgraphs.  Additional iteration to your test to enable stack logging.  1 memgraph with `pre`, another memgraph with `post`.  Can analyze growth if needed.

Now you're ready to understand why regressions occurred.

# Types of memory issues

* Leaks
* heap sizes issues
	* Heap allocation regressions
	* fragmentation

[[iOS memory deep dive - 18]]

## Leaks
Process allocates an object and loses all references without deallocating.   

A common way objects leak is via retain cycles.  Process can't access or free either of them.

Be cautious when using unsafe types in Swift
Watch out for retain cycles, avoid circular strong refs
Consider weak references

Output includes detailed view of object graph.  "ROOT CYCLE" => retain cycle.  

Often, find the section of the callstack with symbols for your code.  Leaking object is populated

```swift
```


Retain cycle example.  Both meal plan and menu item objects have references to each other.  Break reference with `weak`.

## Heap size issues

* Heap stores dynamically allocated objects
* Allocating more objects on heap increases footprint
* Remove unused allocations
* Shrink overly large allocations
* Deallocate memory you are finished with
* Wait to allocate memory until you need it

```bash
vmmap -summary post_file
```

Memory usage by region.  `MALLOC_` for heap.  Since memory footprint = dirty + compressed.

"Swapped" is compressed.  

What kinds of objects are contributing to the regression?  

`heap -diffFrom=pre.memgraph post.memgraph`

13MB of `non-object`. 

> In swift, this usually indicates raw malloc bytes.

`heap -addresses=non-object[500k-] post.memgraph`

One object is 13MB so that's a prime suspect.

Several options.

1.  `leaks --traceTree=address memgraph`
	1.  Useful for specific object, without malloc stack logging enabled.  If you're ever working with a memgraph that doesn't, keep this in mind.
2.  `leaks --referenceTree memgraph`
	1.  Top-down memorytree with a guess as what is root.  
	2.  `--groupByType` makes it easier to parse.


`malloc_history -fullStacks memgraph address`

Produces allocation callstack.  Useful when msl enabled, and address is present.

`Data` object is smart enough to automatically deallocate the buffer.

## Fragmentation
Pages are smallest indivisible unit of memory for your process.
Writing anything to a page makes the whole page dirty
Unused stretches of memory on dirty pages that are too small to use for new allocations.
3 contiguous pages.
1.  Allocations begin filling up
2.  Free memory.  However, these pages are still dirty because there's some allocated object on them.
3.  System fills them with incoming allocation.  However, there's no room because free space isn't contiguous.

Allocate objects with similar lifetimes close to each other.  Notice how fragmentation is a footprint multiplier.  50% frag doubled footprint from 2 to 4 pages.

* Aim for 25% fragmentation or less
* Use autorelease pools
* Pay extra attention to long-running processes

`vmmap -summary`.  Default malloc zone end up in the malloc zone.
With MSL though, we care about `MallocStackLoggingLiteZone`, which is where allocations end up under MSL.  Focus on the right zone.

Dirty+Swap frag size shows me how much memory is wasted due to fragmentation.  If I did have issues, I could use allocations track in instruments.  Specifically, allocations list view and see which objects were created and destroyed.

In the context of fragmentation, destroyed objects create free emmory slots, while persisted objects are remaining.  Both worth investigating for fragmentation.

[[getting started with instruments]]

Now that you'v elearned about memory issues, let's review.

1.  When adding a new feature, write a performance test.
2.  Set baseline
3.  Catch regressions
4.  Collect diagnostics

1.  Check for leaks `leaks`
2.  Check for heap regressions `vmmap-summary`
3.  Find regressed object type `heap -diffFrom`
4.  Find clues in reference tree `leaks -referenceTree`
5.  Get object addresses `heap -addresses`
6.  investigate addresses `leaks -traceTree` and `mallco_history -fullStacks`

# Wrap up
* Eliminate memory leaks
* Look out for retain cycles
* Reduce heap allocations where possible
* Reduce fragmentation with good allocation practices

