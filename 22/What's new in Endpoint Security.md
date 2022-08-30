* rich security event stream
* Replaces KAuth, MACF, and OpenBSM audit

[[Build an Endpoint Security App]]
[[System Extensions and DriverKit - 19]]

# new events
* authentication
* login and logout
* XProtect / Gatekeeper

Authentication types
* password
* touch ID 
* cryptographic tokens
* auto unlock

New events are much richer information and provide visbility into auto-unlock with apple watch, etc.

login/logout
* login window
* screen sharing
* SSH
* /usr/bin/login

Substantially beyond the audit trail, gain more comprehensive visibility into system access, including lateral movement, etc.

gatekeeper
* malicious software detected
* malicious software remediated

OpenBSM Audit trail
* deprecated since macOS big sur
* Will be removed in a future version of macOS

# Improved muting
Since macOS catalina, we've supported muting 

* prevent deadlocks
* manage performance impact

Target path muting
* event-specific
* target path
* target process executable image path
```swift
// Mute events operating on /var/log
es_mute_path(client, "/private/var/log", ES_MUTE_PATH_TYPE_TARGET_PREFIX)

// Mute write events to /dev/null
var events = [ ES_EVENT_TYPE_NOTIFY_WRITE ]
es_mute_path_events(client, "/dev/null", ES_MUTE_PATH_TYPE_TARGET_LITERAL,
                    &events, events.count)
```

Mute inversion (that is, soloing)
Select events intead of muting events
* process
* path
* target path

```swift
// Invert muting for target paths
es_invert_muting(client, ES_MUTE_INVERSION_TYPE_TARGET_PATH)

// Select only events pertaining to /Library/LaunchDaemons
es_unmute_all_target_paths(client)
es_mute_path(client, "/Library/LaunchDaemons", ES_MUTE_PATH_TYPE_TARGET_PREFIX)
```

# Introducing eslogger
Many requests for capability for providing events without writing a native client.  In ventura you can use CLI.

eslogger taps into the event stream for specific events, and makes json-format available to stdout or oslog.  Data is structured just like C representation native clients use.

Hope that helps

* ships with the OS
* Requries superuser privileges (root)
* Requires tCC full disk access authorization

applications continue to interface natively
* schema stability
* performance
* full feature set

## Demo

```bash
sudo eslogger openssh_login openssh_logout >out.jsonl
```

Use jq to examine events.

We look forward to seeing your security solutions, etc.
* https://developer.apple.com/forums/tags/wwdc2022-110345
* https://developer.apple.com/forums/create/question?&tag1=99&tag2=524030
* https://developer.apple.com/documentation/endpointsecurity
* https://developer.apple.com/documentation/endpointsecurity/monitoring_system_events_with_endpoint_security
