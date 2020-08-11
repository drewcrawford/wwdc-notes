#logging

Logs help you understand bugs you can't reproduce.

# Demo
# New logging APIs in xcode 12
Record events as they happen
Archived on device for later retrieval
Low performance overhead
```swift
import os
let logger = Logger(subsystem: category:)
```

`subsystem` is typically bundleID
category is typically user-defined subsystem

You can now use string interpolation.  This is similar to calling the print function.  However

# Log message construction is fast
Not fully converted to string as that is too slow
Optimized based on the type of logged data.

With the optimized version, you only pay the cost of converting to string when the log is displayed.

## Logging types supported
* Numeric types like `Int` and `Double`
* ObjC objects with `-description`
* Any type conforming to `CustomStringConvertible`

Be aware that a nonnumeric type will be redacted in the logs by default.  This is done to ensure that after your app ships, the logs do not show any personal information.  For instance, here I am logging a message with an accountNumber.

However, data that does not contain any sensitive information can be made visible.

When your app logs a message, it is stored on device in a compressed form.

First, connect device
Second, `log collect --device --start 'timestamp' --output fruta.logarchive`

Can also open logs in Console.app

## Stream logs wile app is running
In the console app for a device connected to your mac
Will also see in xcode console when launched with product->run

# Log levels
* Debug - Useful only during debugging
* info - helpful but not esential for troubleshooting
* notice (default) - essential for troubleshooting
* error - error seen during execution
* fault - bug in program

## Message persistence
Only logs that are persisted can be retrieved after execution
Logs that are not persisted can only be seen while the app is running
Persistence is determined by the log level.

Persistence increase with level

* Debug - Not persisted
* info - Mostly not persisted, except when they are generated a bit before `log collect`
* notice (default) -  persisted up to a storage limit
* error - persisted up to a storage limit
* fault - persisted up to a storage limit

Note that `error` and `fault` are persisted longer than notice.  Typically, persisted for a few days, but it depends on storage space on your device.

Log levels also affect performance.  Even though logging in general has low overhead, each level has different performance

debug - most performant
fault - slowest

## Debug levels are very fast
Compiler optimzes away debug messages except while streaming
Safe to call slow functions at debug level, e.g. `logger.debug("\(slowFunctioN(data))"`

Swift compiler uses sophisticated optimiizations, to ensure that the code is not executed when debug is discarded.

# Format data to improve readability
Raw data is often difficult to understand and process
Logging APIs enable formatting data at no run-time cost

`format` and `align` parameters.  Many formatting options.

# Personal data must not be public
Messages are logged even when your app is deployed
Logs can be viewed with physical access to the device
Personal information must not be marked `.public`

Use an equality-preserving hash.  `privacy: .private(mask: .hash)`

New APIS available in ios 14.  Staring in this release, `os_log` also takes format stuff.

