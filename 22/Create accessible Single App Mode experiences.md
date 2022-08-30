How to create accessible single-app mode experiences.  

* Lock the device to a single app
* Initiated by the system or by your app
* Additional restrictions available
* Consider accessibility use cases

Suppose customers are ordering food or drinks.  Or patients handling device.  Faciliating tests in a classroom.

Helps createa  focused environment for the user.  Did you know your app might be used in single app mode if you didn't write code for it?  Guided access lets users put any app in single app mode.

# Guided access
Make sure i'ts turned on in accessibility settinsg.  Open the app you want to logon to.  Triple click to perform shortcut.  Shwos guided access workspace where you configure system restrictions.

Device in a restricted state.  Locked to the frontmost app, restrictions in the options menu are applied.  Guided experience for peopel with cognitive disabilities.

Just perform the accessibility shortcut again.  
* touch
* motion
* keyboard
* volume button
* sleep/wake button

Helps people with cognitive disabilities.  But also parents, etc.

UIAccessibility API lets you create custom restrictions
* tailor the experience
* Restrict or adjust areas of your app
* Accommodate cognitive disabilities


Design principles
* Be forgiving of errors
* Warn before irreversible actions
* Reduce dependence on timing
* Always confirm payments
* Promote user independence

Lock the account settings while guided access is active.  Prevents users from getting lost, etc.  Where they may make unintended changes.  Reduces the amount of times someone gets stuck.

1.  Conform to `UIGuidedAccessRestrictionDelegate`
2. Provide an array of identifiers for each restriction
3. Give a user-facing title for each
4. optional restriction to provide more details
5. Implement `guidedAccessRestriction(withIdentifier:,didChange:)`.

Custom restrictions
* check if enabled: `UIAccessiblity.guidedAccess...`

# Single app modes (programmatically enter)
You're programmatically entering this mode.  Plenty to talk about for setting up, starting, customizing the session.

All benefit from locking the device to a single app.  Modifying things in settings, looking up things in safari, etc.

## Single app mode
The right solution when you have a device that you intend to stay in a single app in perpetuity.
Remains locked after reboot
No manual intervention needed

Device **must** be supervised
Lock a high voume of devices

Apple configurator.  Putting them in single app mode.
Select a supervised device, then advanced=>start single app mode.
#configurator 

Must end through your management software.

## Autonomous single app mode
Restricted state entered and exited often.
Manual intervention between restrictions
API method call to get in or out
Device must be supervised
App must be allow listed (configuration profile) or the function will fail.\

`UIAccessibilty.requestGuidedAccess...`

Guided access serves as the foundation for other single app modes to exist.

You'll want to properly address completion handler result when something goes wrong, etc.  Give an alert to the user and hold off on continuing the experience.

* check if autonomous single app mode is enabled
* `guidedAccessStatusDidChangeNotification`
* Remember that the app which wants to use this API must have supervision and management, including allow-liusting the bundle ID.
## Assessment mode
Prevent unfair advantages by restricting specific features during testing
Unified API for iOS and macOS
Device does **not** need to be supervised
Apply for assessment entitlement
See docs

[[What's new in assessment]]

# Accessibility API
Keep in mind that peopel using assistive technologides do use your apps.

In oru classic example of an iPad being used as a restraurant kiosk.  If device is plainly locked into single app mode, you can't turn on voiceover for the customer.

Apple Configurator and other device management software help configure options.  A handful of accessibility features are available for you to have always enabled.  Accessibility shortcut is configurable to let users quickly toggle accessibility features.  When on or assigned to the accessibility shortcut, people can use them.

Those options have to be configured *before* single app mode.

* Use API on UIAccessibility to toggle features with code
* Alternative to accessibility shortcut
* Housekeeping between transactions
`.configureForGuidedAccess(features:...)`

zoom, voiceover, invertColors, assistiveTouch, greyscaleDisplay.

# Wrap up
* guided access
* single app mode
* autonomous single app mode
* assessment mode
* accessibility API
* https://developer.apple.com/forums/tags/wwdc2022-10152
* https://developer.apple.com/forums/create/question?&tag1=137&tag2=141&tag3=2&tag3=421030
* https://developer.apple.com/documentation/devicemanagement/autonomoussingleappmode
* https://developer.apple.com/documentation/uikit/uiaccessibility/1615186-requestguidedaccesssession
