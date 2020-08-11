#xctest 

# Great tests catch bugs, plan for failure
# Set up
`setUpWithError()` now throws.

Leverage launch arguments and environment variables to set up state.

She uses a launch argument to launch the app to a different tab (being tested) first.  This improves the speed of the tests, but also, isolates her tests from issues going on on the default tab.

# Tests: actions
Each test should have a specific goal in mind, reflected in the title of the test.
Minimizing actions make it easier to triage failures later.

The labels of UI elements change often, so I use enums for all string values.  

Factoring common code into helper functions so that multiple tests can use the same codepath.

Model the domain of my app, and then do tests in that domain.  Basically, she creates an 
```swift
open class FrutaUIElement {
    let app: FrutaApp
	let element: XCUIElement
	
}

public class CustomThing: FrutaUIElement {
	func customAction()
}

```

This way I can focus my queries on the subelements of that element, not the unrestricted app universe.

We have a shared framework for our tests.  May also consider a swift package.

# Tests: assertions

Use the optinal message in XCTAssert functions.  When all I have is the result bundle, there's a lot of context missing.  e.g. `3 is not equal to 2`... 2 what?

Leave myself a clue as to why this expression failed.  But I leave out e.g. dates, timestamps, uniqued file paths.

Use the correct assertion.

New: `XCTIssue`.  [[Triage test failures with XCTIssue]]

Async issues: sleep is not ideal.  XCTest has built in retries, but it might not be enough.  `waitForExistence` with a timeout.  This polls to allow to advance early.  

Bang-related problems.  Basically crashing a test is not ideal if it runs for a long time.

`XCTUnwrap` throws an error if we encounter nil.

I always `throw` instead of `assert` from shared code.  That way I can also support negative tests.

[[Triage test failures with XCTIssue]]

`XCTContext.runActivity` provides debug groups.

`XCTAttachment` gathers extra logging which is really helpful in a CI system.

`XCTSkip` - skip tests that aren't relative for the platform
Can also use for functionality is not yet implemented
Failing tests that cannot be fixed for awhile.

# Tear down
Use throwing `tearDownWithError`
Collect additional logging
Reset the environment

