#xctest #xcode 

# Causes of unreliable code
You may run into inconsistency.  Race conditions, environment assumptions, global state, communication with external services.

Tough to track down because they're challenging to reproduce.

One way to diagnose these failures is to run repeatedly. 

# Test repetition

3 modes
* Fixed iterations.  Fixed number of times regardless of status.  Understanding reliability of test suite and helping keep it reliable as new tests are introduced over time.
* Until failure.  Continues to repeat a test until it fails.  Helps reproduce nondeterministic test failure
* Retry on failure - retry tests until they succeed up to a specificied max.  Useful to identify unreliable tests which fail initially but eventually succeed.  If a test in CI is showing this, can retry temporarily.

Important to remember retryign on failures can hide problems in the actual product.  Best to use this mode temporarily to diagnose failures.

"Run "\<test>" repeatedly" in interactive mode.  To reproduce issues with intermittent tests.
	
Xcode automatically selects "pause on failure" for me, so I can catch the error in the debugger.

Transform this test to use #async await

```swift
	func testFlavors() async throws {
    var truck: IceCreamTruck?

    let flavorsExpectation = XCTestExpectation(description: "Get ice cream truck's flavors")
    truckDepot.iceCreamTruck { newTruck in
        truck = newTruck
        newTruck.prepareFlavors { error in
            XCTAssertNil(error)
        }
        flavorsExpectation.fulfill()
    }

    wait(for: [flavorsExpectation], timeout: 5)
    XCTAssertEqual(truck?.flavors, 33)
}
```

```swift
func testFlavors() async throws {
    let truck = await truckDepot.iceCreamTruck()
    try await truck.prepareFlavors()
    XCTAssertEqual(truck.flavors, 33)
}
```

I'm now confident that this is fixed and ready to remove retry on failure and commit my change.

Another way to run your test with repetition.  Add xcodebuild flags which override any test flag settings.

`-test-iterations <number>
-retry-tests-on-failure
-run-tests-until-failure`

`-test-iterations 100
-run-tests-until-failure`

# Wrap up
Use test repeitions as a tool to diagnose unreliable code
[[Embrace expected failures in XCTest]]
[[Meet asyncawait in Swift]]