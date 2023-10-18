#xcode #xctest 

Key terms and concepts.
# Structuring tests

test methods - Individual method which validates source code
Test classes - groups of test methods
test bundle - groups of test classes, either UI or unit
test plan - unit and UI tests.  

| foo       | source code | user-facing behaviors |
| --------- | ----------- | --------------------- |
| unit test | yes         | no                    |
| ui tests  | no          | yes                      |

configurations.  Tell xcode how to setup the runtime environment for your tests.

* location & language
* code coverage
* test repetitions

run destinations
with xcode cloud / build command, multiple run destinations.

Let's say we're using the same test plan.  Lately, I've been working on supporting many languages.  I've created configurations for the languages I want to support.

an individual instance is called a test method run.

# Explore the test report

From a single isntance toa n entire suite.  Multiple configurations, multiple destinations.  The new test report gives you tools to help you understand your test run regardless of the number of tests.

* get the big picture
* explore patterns
* one-stop shop
* Richer failure information

Test summary gives me an overall understanding of what happened in this test run.

common failure patterns -> similar failure messages
longest rest runs -> which test runs are taking longer than the others.

Understand how my test performed.  Get more details about my test run.  Understand what traits it has, etc.

When testing with many run destinations/configurations, my heatmap can digest how tests are doing.  

test activity - lays out tests in a timeline format.
automation explorer - moment of video playback related to selected test activity.  See a full replay of the test.

Scrubber - linear representation of the test run.  Use the scrubber to locate test events, such as taps, swipes, and clicks.

Quickly find interesting moments in the test run, and ensure itneractiosn work as expected.

Click on elements, see bounding boxes, etc.

# Wrap up
* quickly pinpoint test failures
* Find failure information easily
* Fix failure faster

Xcode
Xcode cloud!

