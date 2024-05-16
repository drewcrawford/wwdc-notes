Learn how to improve the security of your Mac app by adopting environment constraints. We'll show you how to set limits on how processes are launched, make sure your Launch Agents and Launch Daemons aren't tampered with, and prevent unwanted code from running in your address space.

* Frameworks and libraries
* Helper tools and apps
* XPC services
* Launch agents, launch daemons, login items
* app extensions

a hostile environment
can you trust the system where your code runs?
could your xpc service be called by an unexpected process to steal its privileges?
unauthorized execution leak keychain data?

parent processes have a huge amount of influence over children.

posix_spawn gives parent ability to control child.  Parent can also limit access to system resources.  This level of control can cause child to load unexpected code.

Procesess place trust in the disk layout.
Attacker can copy a binary to a new location to
* supply unexpected data to your process via relative path
* remove runtime protections
* modify launch agent, inject libraries, etc.

App Sandbox helps ensures taht your processes do only what they need to do
hardened runtime/library validation
gatekeeper/notarization

Our existing protections focus on running processes rather than execution environment.

* ensure your code can only be launched under expected circumstances
# Environment constraints
macOS has various secure technologies.  in ventura we're using environment constraints as well.  New dimension of OS security.  

Sonoma we expand use to 3rd-party apps.

## what are frogs
* Describe code and circumstances of execution
* signature
* how launched
* role on system

examples

* require that OS Proceses run from system volume
* require that OS daemons are launched only by launchd based on sip protected launchd plist
* require that os apps are launched only via launch services as apps
* detect when a third-party launch agent, launch daemon, or login item changes

## should I use?
* can reduce attack surface of any app
* particularly useful if: multiple processes, load code signed by different teams

## launch constraints

Embedded into binary
properties of process
who can be parent
can be responsible for me

self-constraints, parent process constraints, and responsible process constraints.

e.g. set your xpc service to indicate only MyDemo.app is "responsible" (even though as XPC, launched by launchd).

helper we can require certain launcher.

Launchd plist constraints enforce that a plist registered via SMAppService API will only launch described code
useful if launch daemon could be modified by malicious process

Library load constraints
previously you could adopt library validation, or not.  (you/apple).
with load constraints you can be less restrictive, also preventing arbitrary code

**You cannot exclude apple signed code from loading in your process and you must allow your own code**

# Defining constraints

Implicitly, key-value pair decides if constraint is satisfied.  Each key can only appear once per dictionary.

signing identifier, I guess like a bundle ID?
CDHash -> specify a unique hash for the code that should be allowed.
TeamIdentifier -> specific team.

facts indicate specific properties
operators combine facts

and, or, and-array, or-array, in


# Adopting constraints

Consider an app that * controls registering a launch agent
helper tools
framework, xpc service
library from another team

ex let's say the helper tool has a privilege like keychain.  Helper only launched by your app.

Set a parent process constraint on the helper tool!

'Launch constraint parent process plist'.



### Example constraint - 9:35
```xml
<dict>
    <key>$or-array</key>
    <array>
        <array>
            <string>$and</string>
            <dict>
                <key>team-identifier</key>
                <string>M2657GZ2M9</string>
            </dict>
        </array>
        <array>
            <string>$and</string>
            <dict>
                <key>signing-identifier</key>
                <string>com.smith.libraryB</string>
                <key>team-identifier</key>
                <string>P9Z4AN7VHQ</string>
            </dict>
        </array>
        <array>
            <string>$and</string>
            <dict>
                <key>signing-identifier</key>
                <string>com.friday.libraryC</string>
                <key>team-identifier</key>
                <string>TA1570ZFMZ</string>
            </dict>
        </array>
    </array>
</dict>
```

### Example parent launch constraint - 11:02
```xml
<dict>
    <key>team-identifier</key>
    <string>M2657GZ2M9</string>
    <key>signing-identifier</key>
    <string>com.demo.MyDemo</string>
</dict>
```

Security best practice encourage using XPC services to separate privilege in your apps.

Igf your xpc service has a privilege (ex keychain) then need to ensure who can connect to it.

A responsible process launch constraint limits the processes that can be responsible for the constrained process.


### Example process launch constraint - 14:06
```xml
<dict>
    <key>team-identifier</key>
    <string>M2657GZ2M9</string>
    <key>signing-identifier</key>
    <dict>
        <key>$in</key>
        <array>
            <string>com.demo.MyDemo</string>
            <string>com.demo.DemoMenuBar</string>
            <string>demohelper</string>
        </array>
    </dict>
</dict>
```

Users expect background tasks to happen on your behalf.  But if you replace the launch agent, attacker can gain persistent bg execution on behalf of your app

Use SpawnConstraint key in launchd plist.
### Example launchd plist constraint - 14:52
```xml
<dict>
    <key>Label</key>
    <string>com.demo.DemoMenuBar.agent</string>
    <key>BundleProgram</key>
    <string>Contents/Library/LaunchAgents/DemoMenuBar.app/Contents/MacOS/DemoMenuBar</string>
    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <true/>
    </dict>
    <key>RunAtLoad</key>
    <true/>
    <key>SpawnConstraint</key>
    <dict>
        <key>team-identifier</key>
        <string>M2657GZ2M9</string>
        <key>signing-identifier</key>
        <string>com.demo.DemoMenuBar</string>
    </dict>
</dict>
```

Library loading.  IF you link a library, to get notarized with hardened runtime you need `disable-library-validation`.  But now you can get code by anyone.

To address this, adopt library load constraint.
### Example library load constraint - 15:29
```xml
<dict>
    <key>team-identifier</key>
    <dict>
        <key>$in</key>
        <array>
            <string>M2657GZ2M9</string>
            <string>P9Z4AN7VHQ</string>
        </array>
    </dict>
</dict>
```

# Where available?

* any macOS version
* enforced starting in sonoma
* require a minimum DT of 13.3 and are enforced in 13.3
* facts, operators, and enumerated values can be added later
* see documentation for complete availability info
# wrap up
* process relationships
* launchd plist targets
* library loading


# Resources
* https://developer.apple.com/documentation/security
* https://developer.apple.com/security/
