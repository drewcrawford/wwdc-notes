#xctest

# Testing feedback loop
1.  Write tests
2.  Run tests
3.  Interpret results

Depending on your report, you may decide to write more tests.  Or if you have sufficient confidence, you move to the next task.

Having short feedback loops is important.

# Ensuring you always get feedback

## Guarding against test hangs

 * Deadlock
 * Extremely slow progress
 * Bad timeout values
 * Too much work on the main thread

### Execution Time Allowance

When enabled, xcode enforces a time limit on each individual test.  When it exceeds,

1. capture spindump
2. Kill test that hung
3. Restart test runner

By default, a test gets 10 minutes.  If a test successfully finishes, the timer will be reset fo the next tests.

Customize the default allowance in test plan.

`XCTestCase.executionTimeAllowance` - bounded to the nearest whole minute.

### Customizing default time allowance
* test plan setting
* xcodebuild option

There's a precedence inheritance order.
1.  executionTimeAllowance (highest precedence)
2.  xcodebuild
3.  test plan setting
4.  system default of 10 minutes (lowest precedence), will be overridden

What if test requests unlimited time?

### Enforce a maximum allowance

This basically prevents a test from requesting an unlimited time.

### Recommendations

Use `executionTimeAllownace` to guard against test hangs
Use XCTEst's performance APIs to detect performance regressions

[[getting started with instruments]]

# Get faster feedback

Non-distributed testing: each test will be executed serially on a run destination.

Parallel distributed testing – xcodebuild will distribute tests to each run destination "by class".  It's very important to note that the allocation of test classes to run destinations is non-deterministics.

| os                                                | unit tests | ui tests |
|--------------------------------------------------|------------|----------|
| macOS                                            | ✅          | no       |
| iOS & tvOS simulator                             | ✅          | ✅        |
| iOS & tvOS devices (xcodebuild only) new in xc12 | ✅          | ✅        |


`-parallel-testing-enabled YES -parallelize-tests-among-destinations`
With just 2 devices, XCTest own test suites have a speedup of 30%.

## Recomendations

Ideally, use all the same kinds of devices and OS versions per test suite run.
If using different devices and OS versions, prefer running tests that are device and OS-agnostic.  

To itentionally test against more devices and OS versions, use Parallel Destination Testing.  Destination testing runs the entireity of a test suite

[[what's new in testing - 18]]

