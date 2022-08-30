Autonomonous and Proactive.
[[Meet declarative device management]]

autonomous => reacts to its own state changes
proactive => status channel, asynchronously reporting to the server when important state changes occur.

two key elements to the declarative device management data model.  Declarations and status.  

We have created this technology to enhance management strategies, devices, repetitive and tedious tasks, etc.  To be the driver in their own management state.

* Lightweight and reactive servers
* Monitor devices more closely without polling
* Focus on value-add feature development

Organization benefits
* reliable device state
* Improved clarity and efficiency
* More responsible for users

[[Meet declarative device management]]

# Expanded scope
originally only iOS with user enrollments.

Now avialable for every enrollment type MDM supports.  ADE, Profile-based device rollment, and user enrollments.  Shared iPad.

Tapping the configurations row reveals details about the active configurations.  I'm pleased to announce that declarative device management is available on every platform.

Now supported on macOS and tvOS.

On macOS, configuration section is present in MDM details view, revealing the active configurations.  

Both macOS and shared iPad devices have 2 mdm channels.  These are the device and user channels.    Declarative management must be enabled separately on each channel.

Status reports are separately generated for each channels.
# Status reports
* incremental reports of subscribed status
* Status reports are reliable
* Makes the device proactive

* Passcode
* accounts
* MDM installed apps

## Passcode status
In iOS 15, we introduced a passcode policy configuration.  Tehre can be some lag between policy being applied and the passcode becoming complaint when changed by the user.

MDM servers have to poll the device to determine when the passcode becomes compliant.  There is no need to do that.

We have added two status items.
`passcode.is-compliant` and `passcode.is-present`.

Initially, teh password is likely not complaint so the profile cannot be sent.  Eventually the user changes the passcode to make it compliant.  Server can detect this.

Server sends both the passcode policy and profile, etc., with the profile tied to an activation etc.  Passcode configuration is immediately activated and applies a strong password policy.  Initially, it's likely non-complaint, so it evaluates to false adn the wifi configuration is not activated.  Moved business logic from the server to the device to get more resonsive and reliable device behavior.

```json
{
  "management": {
    "client-capabilities": {
     "supported-payloads": {
       "status-items": [
          ...
          "passcode.is-compliant",
          "passcode.is-present",
          ...
        ]
      }
    }
  }
}
```

## Account status
In iOS 15, we introduced account configurations to isntall accounts of various types on device.  Typically organization accounts, giving the user access to organization data.  It is useful for the admint o know when accuotns have been successfulyl installed and what state they are in.

```json
{
  ...
      "status-items": [
          ...
          "account.list.caldav",
          "account.list.carddav",
          "account.list.exchange",
          "account.list.google",
          "account.list.ldap",
          "account.list.mail.incoming",
          "account.list.mail.outgoing",
          "account.list.subscribed-calendar",
          ...
        ]
  ...
}
```

won't include accounts created manually or via MDM profiles.  Each item corresponds to an account configuiration type, with incoming/otugoing mail separately.

Represent the status of the account type.  Here for example, we have incoming fail.  

Value of the identifier key is a unique identifier for an object within the array of status item objects.

Value of declaration identifier key matches the identifier property value... making it easy to cross-reference the status and configuration.


```json
{
  "identifier": "592D763E-C15B-44F8-A1FC-F88EB1901646",
  "declaration-identifier": "BF8FD199-467B-4BA5-886D-D82B7849E517",
  "hostname": "mail.example.com",
  "port": 443,
  "username": "user01",
  "is-mail-enabled": true,
  "are-notes-enabled": false
}

{
  "identifier": "592D763E-C15B-44F8-A1FC-F88EB1901646",
  "declaration-identifier": "BF8FD199-467B-4BA5-886D-D82B7849E517",
  "calendar-url": "https://holidays.example.com/country/US.ics",
  "username": "user01",
  "is-enabled": true
}
```

* status items now includes array values
* Array items are JSON objects
* Objects always have an identifier key
* Accept unknown keys in objects
* Array values are reported incrementally

Here the server sends to mail configurations to the device.  Then subscription.

Status for the accounts is collected, device sends status report.

Each array object has an identifier.  Server has status for two mail accounts, matching what's on the device.

When server adds a mail account, status item on the device has a new object added to tis array value.  And a new notification is sent to the server.  Only reporting the new item.  Does not match existing items so it corresponds to a new account.

Now server has status for 3 accounts.  When account changes, you get notified about that.  Know it's update based on id value.

Deletions.  Status item on the device has the corresopnding object marked for removal.  Status report sent.  This time you get `removed:true`.  And identifier key.  No other keys.

After processing this report, server has status for two mail accounts, correctly matching the device.
* status is rate limited for optimal performance
* Changes are aggregated over a variable interval (up to 1m)
* Status is reported quickly but not always immediately

## MDM installed apps
* servers poll using MDM commands
* Instead, device can proactively send app isntall state
* Use declarative device management status reports

```json
{
  "management": {
    "client-capabilities": {
     "supported-payloads": {
       "status-items": [
          ...
          "mdm.app",
          ...
        ]
      }
    }
  }
}
```

Only apps installed by MDM are reported, **even on supervised devices**

example object
```json
{
  "StatusItems": {
    "mdm": {
      "app": [
        {
          "identifier": "com.apple.Pages",
          "name": "Pages",
          "version": "7358.0.134",
          "short-version": “12.0",
          "external-version-id": "844362702",
          "state": "managed"
        }
      ]
    }
  },
  "Errors": []
}
```

Indicates the current install phase for the app.  Values of the state correspond to items in MDM response.

server has enabled declarative device management and sent description.  Next step is to install an app using MDM install application command.  Since this is a user enrollment, user approval is needed.  Paused waiting for user input: `prompting`.

Another status report will be sent with app state of `installing`. eventually the app completes installation and is ready for use, `managed`.  

Suppose the user deletes the app.  `managed-but-uninstalled`.  The app is no longer installed, but its management state is being tracked on device.  Let's assume server wants to remove management state.  Sends `RemoveApplication` command to device.  That removes the internally-maintained management state.  App will be removed as well.

# Enhanced predicates
## Activation rpedicates
* Determine whether configurations are applied
* Can reference status items
* Activations re-evaluated when status items change
* Use NSPredicate syntax
* Status item names need special treatment
* Extension term @status used for status items
* `@status(device.identifier.serial-number) == "ABC-XYZ"`
* Previous syntax is deprecated

## Predicates and array values
* status item values can be arrays
* Predicate an activation on an array item
* "Is app com.example.app installed and managed on the device?"

```
SUBQUERY(@status(mdm.app),
         $app,
         ($app.@key(identifier) == "com.example.app") AND ($app.@key(state) == "managed")
        ).@count == 1
```

Status used as the first argument to the subquery.

Subquery expression returns an array of elements taht match the predicate in the third argument.  The `@count` returns the length of the array and returns if result matches.

In order to reference keys, the `@key` extension term must be used.

We will discuss how to use this for new data.

## Manage properties
* servers need to directly control predicates
* Turn complex logic into a simple state change on device
* Assignments, devicer replacements, data protection

New declaration to allow server to set properties on device that can be used in activation predicates.
```json
{
  "management": {
    "client-capabilities": {
     "supported-payloads": {
       "declarations": {
         ...
         "management": [
          ...
          "com.apple.management.properties",
          ...
         ]
         ...
        }
      }
    }
  }
}
```

* JSON object keys defined by server
* Values can be any JSOn value type


```json
{
  "Type": "com.apple.management.properties",
  "Identifier": "AAE09D73-6EF6-4F3B-9E15-11B0F86D5591",
  "ServerToken": "AB4C5B91-3E08-4D4E-A9FF-1E44FE5BFDD4",
  "Payload": {
    "name": "Student One",
    "age": 7,
    "roles": ["grade1", "spanish"]
  }
}
```

Activation with a predicate
```json
{
  "Type": "com.apple.activation.simple",
  "Identifier": "076F928B-9D8E-4BA2-AD34-5655805C82D7",
  "ServerToken": "4FFA91BF-85AE-4053-B8FE-B1C3E507A9CB",
  "Payload": {
    "StandardConfigurations": [
      "3BBB4407-238A-44B1-ABB1-5E7AB95160E0"
    ]
  },
  "Predicate": "(@property(age) >= 18) AND ("Grade12" IN @property(roles))"
}
```

Each property is referenced using @property extension term, with property key inside the term.

* Multiple declarations can be sent
* key names should be unique
* Duplicate keys lead to unpredicatble results (one value arbitrarily chosen)

A school with a set of teachers.  Two divisions, upper and lower. EAch division has its own campus with own wifi network.  Some teachers are admin.  Some teachers are sports coach.  Etc.  Multiple roles, multiple roles for one user, etc.

With traditional MDM, server waits for device assignment.  Server determines teacher roles.  Server determines what profiles for each role.  Server installs each profile.  Changing roles rqeuires adding or removing profiles.Time consuming, slow at peak times.

With new management properties declaration,w e have a more efficieint alternative
* preload declarations ahead of time
* activations predicated on management properties
* Server sends only management properties
* Reduce traffic and complexity

When initially loaded, all predicates evaluate to false.  On day of assignment, all server needs to do is create management properties declarations customized to each teacher.

Preloaded activations are re-evaluated when properties change.

# Wrap up
Declarative device management in iOS 16, tvOS 165, and macOS ventura
status items for passcode, accounts, and MDM installed apps
Enhanced predicates with management properties

NOw is the time to add this to your products.  We're excited to learn how you'll use it.

* https://developer.apple.com/forums/tags/wwdc2022-10046
* https://developer.apple.com/forums/create/question?&tag1=93&tag2=497030
* https://developer.apple.com/documentation/devicemanagement

