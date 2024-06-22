# Debugging model

Learn how to use LLDB to explore and debug codebases. We'll show you how to make the most of crashlogs and backtraces, and how to supercharge breakpoints with actions and complex stop conditions. We'll also explore how the “p” command and the latest features in Swift 6 can enhance your debugging experience.

Debugging as a search problem.
Somewhere between program start and incorrect behavior, faulty code was executed.

Inspecting state of program at different points in time.  Each inspection brings us closer to the problematic code.  Log files are one way to inspect program at points in time.  Requires foresight by the programmer to guess what is useful to log.

Print debugging.  Need to remove print statements.  How to navigate search space faster by using debugger.  Main tools provided by lldb.

* backtraces
* variable inspection
* breakpoints
* expression evaluation

Running the program, breaking, inspecting program state.  After inspecting the program, we can proceed to a later point in execution.  Repeating the run, break, inspect loop efficiently is key to an effective debugging session.



# Starting the debug session
`lldb -- <executable> <arguments>`

On apple platforms, whenever a program crashes, crashlog is created.  lldb is able to consume crashlogs and present in a form that resembles a debugging session.

xcode uses lldb to create a debugging session with a state of the program at the time of the crash.  Code is failing to open a json file.  Immediately before the crash, the program logged the filename that it was trying to open.

Crashlogs work best when:
* repository on the same commit as the version of the app that created the crashlog
* The dSYM bundle for that build is available
[[Symbolication Beyond the basics]]


# Breaking
`break list`

lldb assigns an id to each location.  ex, 1.1.  Same identifier used by xcode when it highlights the bp line.

closures have their own breakpoint.


# Inspecting program state

### 8:09 - WatchLater button
```swift
Button(action: { watchLater.toggle(video: video) }) { 
    let inList = watchLater.isInList(video: video) 
    Label(inList ? "In Watch Later" : "Add to Watch Later", systemImage: inList ? "checkmark" : "plus") 
}
```

1.  Constructor call
2. trailing closure (called by constructor)
3. action closure (button clicked)

Let's try to learn how UI elements interact with the program and focus on the first breakpoint.

To break only when the button is created, disable the last two button locations.


### 12:54 - Printing watch later list information
```swift
p watchLater.count 
p watchLater.last!.name
```

### 13:45 - Breakpoint actions: Printing name of the most recently added video
```swift
p "last video is \(watchLater.last?.name)"
```

## On the command line
Continues execution after printing.  This command affects the most recent breakpoint, but it can modify another breakpoint by argument.
### 14:42 - Breakpoint actions: on the command line
```swift
b DetailView.swift:70 
break command add 
p "last video is \(watchLater.last?.name)" 
continue 
DONE
```
`help command <option>`.
Consider `apropos <keyword>`.  Returns any commands or options described by that keyword.

## high-firing breakpoints

3 main techniques. Conditions:

`break modify <ID> --condition "video.length > 60"`

note that actions can be used to create new breakpoints

`tbreak Importer.swift:67` -> stop only once at the location.

Breakpoint ignore count - `break modify <ID> --ignore-count 10`

In extreme cases these may slow down program execution.  Recompiling code may be advisable.
Consider `raise(SIGSTOP)`.  Instructs the application to stop, and xcode/lldb we take over.

# Inspecting program state

p, po, registre read, memory read, target variable, etc.

since xcode 15, p is the right command for most situations where you need to inspect a varaible or evaluate an expression.

It's an alias to `dwim-print` ("do what I mean").
[[Debug with structured logging]]

Swift error breakpoint.  

With Swift 6, the output of `p` can be customized with `@DebugDescription` macro.  We must also create a `debugDescription` string property summarizing the type.

If you have used `CustomDebugStringConvertible` protocol before, it works via `po`.  If you're only using string interpolation and computed properties, you can adopt the macro instead.  Those types integrate much better for the debugger. 


### 26:46 - @DebugDescription macro example
```swift
// Type summaries 
@DebugDescription 
struct WatchLaterItem { 
    let video: Video 
    let name: String 
    let addedOn: Date
    var debugDescription: String { 
        "\(name) - \(addedOn)" 
    } 
}
```

# Wrap up
* debugging is a search problem
* LLDB accelerates the search for bugs
* lets you explore codebases faster
* lets you use programming skills to debug your program
* can execute code without recompile!

# Resources
* https://lldb.llvm.org/
