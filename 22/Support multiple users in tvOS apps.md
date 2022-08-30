#tvOS 

Excited to share improvements to tvOS 16 to make it easier to support multiple users in your app.

Apple's TV is designed to be used by everyone.  For multipel users, we have new features taht will make it easier to support multiple users in all apps.

Overview of multiple users features on apple tv, showing how easy it is to 
# personalize tvOS apps
List of usrs includes family members that haven't been added yet, making it easier to add them. + icon indicates a suggested user.  Bring their iPhone to the same room.  On the iPhone, confirm the connection.  That's all.

App running on eprsonal device can simply store the preferred profile in NSUserDefaults, or even put it in cloudkit to sync to all devices.  iPhone of each member of the family remembers the person's preferred profile.

In tvOS 14, we introduced ability to run as the current user.  Single checkbox to add "runs as current user" entitlement.  Apps can access each users' own data just like on iPhone.

iOS code can run as-is on apple tv.  Behaving like each person is using their own personal TV.  tvOS takes care of privacy, security, etc.  Runs with current user entitlement.

Signing in needs to be as easy and infrequent as possible.  We itnroduced a feature that allows peopel to use their iPhone/iPad to sign into tvOS app.  First-class sign-in experience.  Lets you access iCloud keychain etc. which isnt' available on apple tv

[[Simplify sign in for your tvOS apps]]

This year, we're introducing OAutha and PassKeys on tvOS.  

[[Meet passkeys]]

To achieve this optimal user experience, we're introducing a simple new API in tvOS 16.  

Access keychain items across users.
```swift
func save(username: String, password: String) {
    guard let passwordData = password.data(using: .utf8) else {
        return
    }

    let attributes: [CFString: AnyObject] = [
        kSecAttrService: "MyApp" as AnyObject,
        kSecClass: kSecClassGenericPassword,
        kSecAttrAccount: username,
        kSecValueData: passwordData,
        kSecUseUserIndependentKeychain: kCFBooleanTrue
    ]

    let status = SecItemAdd(attributes as CFDictionary, nil)
    if status == errSecSuccess else {
        self.credentials = (username, password)
    }
}
```

these items are always accessible by all users.

With useri ndependent keychain set, items can be readable/writable by all users.

* adopt "runs as current user" entitlement
* store the user's preferences using APIs like NSUserDefaults
* store login credentials in the user-independent keychain

We've deprecated the methods to manually map profiles to system users.  No need to mantain a map of users to profiles anymore.  System handles this.  Same APIs from iOS can be used as-is on apple tv.

Add capability for "User Management".  Then I can get the privilege "run as current user".  In tvOS 16 we can maintain the experience of having a single account for all users, by running as the current user.

Use user defaults to remember the profile section for each user.  Now that my app runs as the current user, apple tv can be as personal as iPhone, use the same code, etc.

System will put up a UI while it gives your app time to finish tasks.  



# demo

# Recommendations
Let's review how apps work on apple tv.
Apps **without** the entitlement, runs as the "default user".  You can think of this like the apple tv itself as a user.  Switching users has no influence on the app's process.

|                          | xcode                                                                        | multiple users | shared login |
| ------------------------ | ---------------------------------------------------------------------------- | -------------- | ------------ |
| media apps with profiles | Add "runs as current user" entitlement, use kSecUseUserIdendependentKeychain | yes            | yes          |
| Per-user apps            | Add "runs as current user" entitlement                                       | yes            | no           |
| All others               | Default behavior                                                             | no             | yes             |

# Wrap up
* direct content access
* Keep dialog prompts to a minimum

[[Support multiple users in your tvOS app]]

* https://developer.apple.com/forums/tags/wwdc2022-110384
* https://developer.apple.com/forums/create/question?&tag1=246&tag2=576030
* https://developer.apple.com/documentation/tvservices/mapping_apple_tv_users_to_app_profiles




