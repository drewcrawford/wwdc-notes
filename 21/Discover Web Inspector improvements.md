#webkit 

Access from develop menu
# Grid overlays
CSS grid is a 2-dimensional layout system.  Design layout that's incredibly powerful and instantly familiar.

Two righthand panels.  One has the "authored" style, the way it is inherited in CSS etc.

Clickable grid badge.  If we click this, we get a new grid overlay on top of your existing content, letting you visualize inline.  

Web inspector.  Layout panel.  Shows all options and controls related to grid overlays.

1.  Grid overlays on/off
2.  Single overlays off/on

Options.

1.  Track sizes => row or column of your grid
2.  Line numbers => values to place child elements at specific rows or column.  negative line numbers count from the last explicit line.  
3.  Line names provide another ways to refer to lines.  
4.  Area names let you see each named area.  
5.  Extended gridlines allow you to see tracklines extended to edge of page.  

Gridlines are very fast.  Right-to-left, left-to-right

iOS 15 and iPadOS 15 when inspecting remotely from a mac with latest safari or tech preview.

## Solve problems demo
`grid-area: 4 / 6;`
row track line 4, column track line 6.

It looks like negative lines are from the right or trailing?  So we can relate from either side?

# Breakpoints enhancements
So much more to breakpoints.  In fact, 5 types.
1.  Debugger statements
2.  Exceptions
3.  assertions
4.  JS breakpoints
5.  Event breakpoints
6.  DOM breakpoints
7.  URL breakpoints

JS breakpoint, right click => edit.  
1.  Condition `table === $0`.  New this year, refer to selected DOM node etc.
2.  Ignore n times before stopping
	1.  When used in addition, count is incremented only if condition is satisfied
3.  Action
	1.  Evaluate JS
4.  Log message
5.  Play sound
6.  Probe expression which is evaluated and put into the "probe" panel.
7.  Automatically continue after evaluating

Actions that support JS can be set to "emulate user gesture".  Quickly test new behavior before implementing the change in webpage source

Mix-and-match action types

Now supported for all breakpoint types.  

## Demo
Modify breakpoint not to break on every click.

In this case, I can use web inspector console to `$event.target === $0`

Identify and test without console.log. 
# Audit authoring
Create and edit tests right from Web Inspector.

By default, there are 2 test groups
* Demo => tour of how audits work
* Accessibility => check for subset of DOM best practices for ARIA

To share tests, export

Make changes to the resulting JSON file, and import modified tests back into web inspector.  

Setup function for all tests at/below a level

Return true/false or a result object

Learn more about the API in the Web Inspector reference.

## Demo
Quickly run these tests in the test group.

Open "Edit" to start editing the audits.
Create a new testcase using the "Create" button at the top pof the sidebar.

Export tests.  

# Wrap up
* webkit.org
* Introducing CSS Grid Inspector
* webkit.org/web-inspector
* Feedback assistant

[[Design for Safari 15]]
[[Develop advanced web content]]
[[What's New in Web Inspector]]

* https://developer.apple.com/safari/download/
* https://developer.apple.com/documentation/safari-release-notes
* https://webkit.org/web-inspector/
* https://developer.apple.com/bug-reporting/

