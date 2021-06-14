Improvements to breakpoints.

* Inspect process state
* Follow process execution
* Pause the bug before it happens
* Breakpoints make process pause

Best way to pause is to use a breakpoint.  

# Source file breakpoints
Set in a single file
line breakpoint
click in the gutter.

When I step in, I'm actually stepping into a different expression.  Compiler has determined that an expression has to be executed first.

Repeating stepin/stepout can be laborious.  Sometimes a line breakpoint is not granular enough.  More than one location for lldb to stop.  What we really want to do is to pause just before a function is executed.

In xc13, we have column breakpoints.  Avoid the shortcomings of line breakpoints.  Click to disable or enable.  

XC draws a highlight over the column.  Green highlight under the next expression (column program counter).  I can confidently do a 1-step into the fn.  

Useful for closures in swift, or cols in objc.  

Compiller creates a table, "line table".  Maps soft lines and columns to compiled addresses.  For each closure on the line, compiler generates a line table entry used to parse in debugger.


# Symbolic breakpoints

* Breakpoints on function names
* Helpful when source file breakpoints are less effective

Can break on many functions at once.  lldb will match the name in every loaded process, including system libraries.  Can be many resolved breakpoint locations. 

Can restrict search to a particular module.  Binary image that can be loaded during execution.  Here we enter our app name.  

For symbolic breakpoints, it's fairly easy to make a typographic error, and then during execution, the breakpoint doesn't hit.  In xc13, if a breakpoitn is not resolved, xcode will show you a dashed icon.

## Unresolved breakpoints
* Tooltip to explain possible reasons
* For symbolic breakpoint, has to be spelled correctly and exist
* Library must be loaded.  Sometimes, the library is only loaded based on user action.

```lldb
image lookup -rn convert <module>
```

This time, lldb has resolved it successfully and given us a location.

[[LLDB Beyond PO - 19]]

Unresolved brekapoints can also be sourcefile breakpoints.  

* Line for the breakpoint must be compiled.  e.g. not `#else` or something.
* Compiler must have generated debugging info for the odule.

# Runtime issue breakpoints

By default, xcode doesn't pause your process.  Instead, when a runtime issue occurs, xcode records the backtrace and presents in the issue navigator.  But since the issue happens in the past, no point in inspecting the current process state.

To catch when it happens, use a runtime issue breakpoint.  Select a specific type by using the type.

For some of them, need to enable the corresponding feature.  Can go there by clicking "go to".

[[Advanced debugging with Xcode and LLDB]]
[[LLDB Beyond PO - 19]]

