#xctest 

Conditions that can only be determined at runtime.

# Conditional test execution
* Unmet requirements
* Return early or fail?
* Both are undesireable

Passing the test suggest it's working when it's not validated, failing may indicate a failure

# XCTSkip
Introduced in Xcode 11.4
New test result
Pass, fail, **Skip**

XCTSkip can be `thrown` evidently?

also `XCTSkipIf()`

# APIs
Throwing functions
* `XCTSkipIf()`
* `XCTSkipUnless()`

Can also throw the `XCTSkip` struct directly.  This can be useful e.g. for `#available`

