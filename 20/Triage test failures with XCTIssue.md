#xctest 

# Understanding test failures
* what failed?
* how did it fail?
* why?
* where did it happen?

[[Write tests to fail]]

Now displays entire callstack.


# Swift errors in tests
Improvements to swift runtime in 13.4 and 10.15.4 make it possible for XCTest to report locations for errors thrown inside tests.  So you no longer need boilerplate.

Also available in `setupWithError` and `tearDownWithError`

# Rich failure objects
Failures:
* Failure message
* File path
* line number
* "expected" flag

In xc12, these values are encapsulated by `XCTIssue`

Adds
* Distinct types
* detailed description
* associated error
* attachment

`XCTAttachment` - capture arbitrary data
add to test or XCTActivity
XCTIssue supports attachments
Custom diagnostics for test failures

New API on XCTestCase 	`record(_ issue:)`

In Swift, `XCTIssue` is immutable.


# failure call stacks

We formerly just had a line number / filename, but this is complicated in the case that there are helper methods involved in tests.  Do we annotate the test or the helper function it calls?

Now, `XCTIssue` captures and symbolicates whole callstacks.


# advanced workflows

You can implement custom assertion by creating your own `XCTIssue` and calling `record(issue)`

You can override `record(issue)` to observe, pass, modify, etc., issues.  If you do not call super, this will suppress the issue.  Can also add attachments via this workflow.

