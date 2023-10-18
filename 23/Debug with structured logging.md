Discover the debug console in Xcode 15 and learn how you can improve your diagnostic experience through logging. Explore how you can navigate your logs easily and efficiently using advanced filtering and improved visualization. We'll also show you how to use the dwim-print command to evaluate expressions in your code while debugging.

# Tour debug console
Can see stuff below log files
Quicklook each log
show/hide logs

# Live debugging
type in filter field and choose popopup to filter
jump to source

```swift
.onSubmit {
  logger.info("Requesting to change displayName to \(displayName)")
  accountViewModel.setDisplayName(displayName)
}
```

```swift
public func setDisplayName(_ newDisplayName: String) {
  logger.info("Sending Request to update DisplayName")

  Database.setValueForKey(Database.Key.displayName, value: newDisplayName, forAccount: account.id)

  logger.info("Updated DisplayName to '\(newDisplayName)'")
}

public func setEmailAddressName(_ newEmailAddress: String) {
  logger.info("Sending Request to update DisplayName")

  Database.setValueForKey(Database.Key.emailAddress, value: newEmailAddress, forAccount: account.id)

  logger.info("Updated DisplayName to '\(newEmailAddress)'")
}
```

```swift 
public func setDisplayName(_ newDisplayName: String) {
  logger.info("Sending Request to update DisplayName")

  Database.setValueForKey(Database.Key.displayName, value: newDisplayName, forAccount: account.id)

  logger.info("Updated DisplayName to '\(newDisplayName)'")
}

public func setEmailAddressName(_ newEmailAddress: String) {
  logger.info("Sending Request to update DisplayName")

  Database.setValueForKey(Database.Key.emailAddress, value: newEmailAddress, forAccount: account.id)

  logger.info("Updated DisplayName to '\(newEmailAddress)'")
}
```

```swift
public func setDisplayName(_ newDisplayName: String) {
  logger.info("Sending Request to update DisplayName")

  Database.setValueForKey(Database.Key.displayName, value: newDisplayName, forAccount: account.id)

  account.displayName = newDisplayName

  logger.info("Updated DisplayName to '\(newDisplayName)'")
}

public func setEmailAddressName(_ newEmailAddress: String) {
  logger.info("Sending Request to update DisplayName")

  Database.setValueForKey(Database.Key.emailAddress, value: newEmailAddress, forAccount: account.id)

  account.emailAddress = newEmailAddress

  logger.info("Updated DisplayName to '\(newEmailAddress)'")
}
```

```lldb
(lldb) po account
```


```lldb
(lldb) po account
<Account: 0x60000223b2a0>
```

```lldb
(lldb) p account
(BackyardBirdsData.Account) =0x000060000223b2a0 {
	id = 3A9FC684-8DFC-4D7D-B645-E393AEBA14EE
	joinDate = 2023-06-05 16:41:00 UTC
	displayName = "Ravi Patel"
	emailAddress = "ravipatel@icloud.com"
	isPremiumMember = true
}
```

```lldb
(lldb) p account
(BackyardBirdsData.Account) =0x000060000223b2a0 {
	id = 3A9FC684-8DFC-4D7D-B645-E393AEBA14EE
	joinDate = 2023-06-05 16:41:00 UTC
	displayName = "Johnny Appleseed"
	emailAddress = "ravipatel@icloud.com"
	isPremiumMember = true
}
```


# LLDB improvements

`dwim-print`.  Use a single comamnd to evaluate many different expressions.

shortcut: `p` now does that.

If you do want the custom object description, you can do `dwim-print + object-description`.
but now `po` is a shortcut for that!


# Tips for logging

stdio is for cmd-line ui
oslog is for debugging (not print!)


convert stdio to oslog


```swift
func login(password: String) -> Error? {
  var error: Error? = nil

  …

  loggedIn = true
  return error
}
```

add print statements

```swift
func login(password: String) -> Error? {
  var error: Error? = nil
  print("Logging in user '\(username)'...")

  …

  if let error {
    print("User '\(username)' failed to log in. Error: \(error)")
  } else {
    loggedIn = true
    print("User '\(username)' logged in successfully.")
  }
  return error
}
```

now we need markers

```swift
func login(password: String) -> Error? {
  var error: Error? = nil
  print("🤖 Logging in user '\(username)'... (\(#file):\(#line))")

  …

  if let error {
    print("🤖 User '\(username)' failed to log in. Error: \(error) (\(#file):\(#line))")
  } else {
    loggedIn = true
    print("🤖 User '\(username)' logged in successfully. (\(#file):\(#line))")
  }
  return error
}
```
logger

"it is common to use a bundle id for subsystem, and a class for category"

```swift
import OSLog

let logger = Logger(subsystem: "BackyardBirdsData", category: "Account")

func login(password: String) -> Error? {
  var error: Error? = nil
  print("🤖 Logging in user '\(username)'... (\(#file):\(#line))")

  …

  if let error {
    print("🤖 User '\(username)' failed to log in. Error: \(error) (\(#file):\(#line))")
  } else {
    loggedIn = true
    print("🤖 User '\(username)' logged in successfully. (\(#file):\(#line))")
  }
  return error
}
```


```swift
import OSLog

let logger = Logger(subsystem: "BackyardBirdsData", category: "Account")

func login(password: String) -> Error? {
  var error: Error? = nil
  logger.info("Logging in user '\(username)'...")

  …

  if let error {
    logger.error("User '\(username)' failed to log in. Error: \(error)")
  } else {
    loggedIn = true
    logger.notice("User '\(username)' logged in successfully.")
  }
  return error
}
```

# Get the most out of your logging
* create multiple log handles, so you can search meaningfully
* OSLogStore to colelct diagnostics in the field
* OSLog is a tracing facility, ex see instruments

```swift
let logger = Logger(subsystem: "BackyardBirdsData", category: "Account")
logger.error("User '\(username)' failed to log in. Error: \(error)")
logger.notice("User '\(username)' logged in successfully.")
```

# wrap up
* explore the new console
* switch from `print` to `OSLog`
* Start with lldb's `p` (dwim-print)


# Resources
https://developer.apple.com/documentation/os/logging

[[Explore logging in Swift]]
[[Unified logging and activity tracing - 16]]
