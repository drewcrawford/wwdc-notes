#safari

Preferences, Show Develop Menu, Then develop->Show web inspector

# Interface
Chrome changes
Dark mode variants for all the icons
Network shows page statistics below the main table

## Sources
View page resources
Debug JS execution
Create and edit overrides

Can now click "Response" breadcrumb to get "Response (DOM Tree)" to visualize as a DOM tree.

Also available for JSON.

Local overrides -> the button in the upper right, left-most.  Editable, and preserved across sessions.

Edit local override... Can also modify the status code, etc., for headers.
When you laod anything that matches, it will be replaced with the local override basically.

Add "inspector bootstrap script".  A way to modify the JS API surface itself.  This allows you to swizzle builtin functions, or setup global state.  Just like local overrides, it's preserved across sessions.

Global breakpoints.  "Debugger statements" controls whether JS execution is paused.  This allows you to avoid turning off "all breakpoints".

### New breakpoint types
* All microtasks - pause whenever any microtask is about to be executed, e.g. promises or queue microtask global function
* animation frames
* timeouts
* intervals
* all events -> any event listener.  

Is a global breakpoint too global?

jquery hacks my event listeners.  How do I only stop in my code?

Script blackboxing.  Defer anything that would happen in a script, to break instead in my code, after the script.  Click the "hide" button next to the script.

Can also blackbox by regex.

Clicking on any line in the gutter will add a JS breakpoint, even after prettyprinting.

```js
let foo = a() || b () || c();
```

How do I break on `c()`?  Step in/out?

"Step" will resume and repause before the next *expression* in the callframe.  Can move from `a` to `b` to `c`.

What about minimified js/html?

Web inspector will now format content, allowing support for all the existing JS debugger and stepping capabilities too.  Pretty-print button is in top-right.

## Timelines tab
Majority of performance profiling tools
Export/import

Bit like instruments, really.

Can "Edit" to get new timelines.  This year, **Media & Animations**.

Correlate activity captured in other timelines.  Creation/destruction of any CSS transition or animation.

## Storage tab

Enumerates data stored in the browser.  Cookies, localStorage, indexDB

Always-visible filter bar.  Can now double-click and edit a cookie.

## Graphics
Expands scope of canvas tab.

Shows previous for all `<canvas>` contexts and shaders
Animations

[[What's new for web developers]]

## Layers
Visualizes the layer tree of the inspected page
Try to keep the number of unintended layers as low as possible.

Basically view debugger for the web.

## Console
Lists all logs from the inspected page
Allows arbitrary JS evaluation
Exposes special engine functionality

`queryInstances(Class)`
`queryInstances(Superclass)` returns children as well
`queryInstances(Type.prototype)` unclear why we do this

`queryHolders(instance)` finds strong pointers to `instance`

Enable Intelligent Tracking Prevention Debug Mode

# General
* Tooltips are your friend.
* Context menus are *everywhere*.  Many *icons* have a context menu.  Every *link* has a context menu.
* Web Inspector Reference
* Safari Technology Preview


