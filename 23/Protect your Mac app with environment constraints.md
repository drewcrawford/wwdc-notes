Learn how to improve the security of your Mac app by adopting environment constraints. We'll show you how to set limits on how processes are launched, make sure your Launch Agents and Launch Daemons aren't tampered with, and prevent unwanted code from running in your address space.

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

# Resources
* https://developer.apple.com/documentation/security
* https://developer.apple.com/security/
