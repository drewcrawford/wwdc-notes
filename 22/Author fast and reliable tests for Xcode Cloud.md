I work on XCTest.  In this session, I'm going to share the most effective ways to start testing for Xcode Cloud.  Our teams designed XC Cloud to be a powerful tool for all developers.

We use it to test xcode itself.  Substantially broaden a given test suite.  By cvonfiguring most tests to run in the cloud, you now have a practical way to run tests on multiple destinations including differing operating system versions. iPhone, iPad, Mac, etc. 

Run various test plan configurations allowing for runtime analysis tools such as address and thread sanitizers.  Once we have passed a thorough test suite,w e can be confident the code is ready to ship.

Offloading tests allows a broader set of tests without impacting desktop cycle of code, compile, and test.  Wtih this expanded test suite, potential for an increased number of unreliable tests.  Ensuring reliability is essential.  Such a large number of tests need to run efficiently so limit their impact on CI process.

# Authoring reliable tests
[[Meet Xcode Cloud]]

* Implement `setUp()` and `tearDown()`.  Tests make use of a fresh simulator which may not meek original assumptions.

* avoid local dates and times.
* Configure locale
* Mock device permissions.
* Prepare dependent files

```swift
var truck: Truck!

override func setUp() async throws {
    let directoryURL = FileManager.default.temporaryDirectory
    let fileName = UUID().uuidString
    let fileURL = directoryURL.appendingPathComponent(fileName, isDirectory: false)
    let data = await mockDonutMenuData()
    try data.write(to: fileURL)
    truck = Truck(menuURL: fileURL)
}
```

Rather than relying on teardown methods, we recommend establishing all state prepartion in the setup method.  In many cases, read only files can be checked itno the repo and accessed by tests.  But when these need to be constructed, we support running a custom build script.

For more details,

[[Customize your advanced Xcode cloud workflows]]

## XCTSkip tests that fail to meet preconditions
instructs XCTest runner to cease executing the current test and mark as skipped.  Used to bypass a yet-to-be supported version or device type.

Also leverage via environment variables.   Parameterize test runner and test host.

In xcode cloud, environment variables prefixed with `TEST_RUNNER_` get passed to the test runner.  This prefix is stripped before you see it.
* `BASE_URL` becomes `TEST_RUNNER_BASE_URL.  
* test plans require the same format as test code, e.g. we do not add the `TEST_RUNNER` prefix.

```swift
var truck: Truck!

func testOrderDonut() throws {
    let host = ProcessInfo.processInfo.environment["BASE_URL"]

    let expectation = XCTestExpectation(description: "Order donut")
    truck.order(with: .sprinkles, host: host) { error, donut in
        XCTAssertTrue(donut.hasSprinkles)
        expectation.fulfill()
    }       
    wait(for: [expectation], timeout: 5)
}
```

Redefining an enviornment variable in multiple places can lead to unexpected results.  e.g. test plan and xcode cloud user interface.  In this partiuclar case, xcode cloud's environment variables take precedence over test plans.  

Cloud reports in lefthand xcode panel, control click on food truck.  Manage Workflows...
Sidebar: environment
Add environment variables.

We could instead set within the test plan.  

Now we have a test plan called "Food Truck".  Click on test plan to open its editor.  Near the top, we can select between "Tests" and "Configurations".  In configurations, we set the environment variables.

```swift
var truck: Truck!

func testOrderDonut() throws {
    let host = ProcessInfo.processInfo.environment["BASE_URL"]
    try XCTSkipIf(host == "prod.example.com")

    let expectation = XCTestExpectation(description: "Order donut")
    truck.order(with: .sprinkles, host: host) { error, donut in
        XCTAssertTrue(donut.hasSprinkles)
        expectation.fulfill()
    }       
    wait(for: [expectation], timeout: 5)
}
```

To learn more about skipping ets, [[XCTSkip your tests]]

## Increase XCTestExpectation timeout or replace with async/await

Test may fail due to unexpected timeout.  ex, slow sever or anxious UX test.

Can increase expectation timeout so that interaction will finish.  

```swift
var truck: Truck!

func testOrderDonut() throws {
    let host = ProcessInfo.processInfo.environment["BASE_URL"]
    try XCTSkipIf(host == "prod.example.com")

    let expectation = XCTestExpectation(description: "Order donut")
    truck.order(with: .sprinkles, host: host) { error, donut in
        XCTAssertTrue(donut.hasSprinkles)
        expectation.fulfill()
    }       
    wait(for: [expectation], timeout: 10)
}
```

vs

```swift
var truck: Truck!

func testOrderDonut() async throws {
    let host = ProcessInfo.processInfo.environment["BASE_URL"]
    try XCTSkipIf(host == "prod.example.com")

    let donut = try await truck.orderDonut(with: .sprinkles, host: host)
    XCTAssertTrue(donut.hasSprinkles)
}
```



Declare tests that are expected to fail.

Failure in a test will be reported as an expected failure etc.

```swift
var truck: Truck!

func testOrderDonut() async throws {
    let host = ProcessInfo.processInfo.environment["BASE_URL"]
    try XCTSkipIf(host == "prod.example.com")

    XCTExpectFailure("<https://dev.myco.com/bug/98> Donut ordering service is down")
    let donut = try await truck.orderDonut(with: .sprinkles, host: host)
    XCTAssertTrue(donut.hasSprinkles)
}
```

[[Embrace Expected Failures in XCTest]]

## Leverage test repetitions
Runs the same test multiple times waiting for
* first failure
* first success
* statistical result

ex
* Confirm initial reliability
* Diagnose unreliable code
* Retry inconsistent services
	* while this is a powerful tests, it's preferrable to mock when possible.
	* determinstic reliability
	* speed

[[Testing tips and tricks]]

Test plan.  Configurations.  Test repeition mode: None.  Retry on failure => work around unrealiable external services.

[[Diagnose unreliable code with test repetitions]]
[[Write tests to fail]]

# Configuring for faster results
A number of configuration options 

Create second workflow containing only pull request tests.
Identify a reduce set of tests to verify as part of each open or update toa  PR.
We could run the unit tests along with just a key subset of user interface tests for a single platform.

Full set of tests on all supported platforms can still be run, but now in the backgroudn and not blocking PRs.  Tests and new platforms while keeping our CI process timely.

We created a new test plan called "Pull Requests" and have it open in the test plan editor.  Tests and configurations.  Select "tests".

Chosen a subset of tests to be verified for a pull request.  To setup a workflow to run our PR test plan, we'll go back to xcode cloud manage workflows just like we did when adding an environment variable for skipping tests.

Now we have a PR start condition.  Runinng tests require that the FT app first be built. Need to add a build action.

In the sidebar, below start conditions, let's add an action.  "Add" button next to actions, and select "build" from context menu.  

Now our workflow is configured to run on PR.  Set the other one to be "on a schedule for a branch".  

[[Testing in xcode - 19]]

Can also enable parallel test execution.  By default, xcode cloud tests your platforms in parallel.  In addition, you can enable xcode to run tests in parallel on a target and test object class level.

Note that tests must be designed to run indepedently.  Proper setup and teardown are essential to reliable test case behavior.

Halt runaway tests by setting execution time allowance.  Number of seconds for a test to run before it fails with a timeout error.  Prevents a test suite from getting stuck on an individual test.  

Configure an execution time allowance for our test plan.  To set the allowance, goto test plan, configurations.  Test timeouts (off), number of seconds, etc.  Default is 600 seconds.

Having configured this, a single runaway test will no longer disrupt our testing workflow.  We are able to move onto next improvement:

* Minimize retries for unreliable tests.

These repeitions can add to the time it taekes to run the test suite.  May want to optimize this to a lower value in test plan.


[[Getting your test results faster]]


# Recap
configuring tests to be both reliable and fast to avoid irrelevant failures and verify code changes quickly.





