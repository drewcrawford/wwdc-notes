# Recent addresses
Will get recent addresses from other apps.  If you're interested in promoting locations from your app,

[[Increase Usage of your App with Proactive Suggestions - 16]]

## from other apps
```swift
let streetAddressTextField = UITextField()
steetAddressTextField.textContentType = .fullStreetAddress  

//Other address granularity: 
// .addressCity, .addressCityAndState, .addressState, .countryName
// .postalCode, .streetAddressLine1, .streetAddressLine2, .sublocality
```
Can also set `UITextContentType` in the xcode attributes inspector.
Be as precise as possible.  You can't combine multiple values for 1 text property.
For navigation app, the `.fullStreetAddress` might be right, but for weather, `.addressCity` might be enough.

# Contact information
But is asking for contact access a good idea?  Users will be prompted to allow access.  Not only will this interrupt their flow, they might not feel comfortable.

Even if they do choose to share contacts, my app has a greater risk for potential privacy exposure.  

`CNContactPickerViewController`.  The app does not need access to contacts and users will not be prompted to grant permission.  App only has access to the specific information that the user chooses to share with the app.

In iOS 14, we are suggesting contacts in autofill.  

```swift
// AutoFill contacts' email address
let emailTextField = UITextField()
emailTextField.textContentType = .emailAddress 

// AutoFill contacts' phone number
let phoneTextField = UITextField()
phoneTextField.textContentType = .telephoneNumber 

// AutoFill contacts' address 
let streetAddressTextField = UITextField()
steetAddressTextField.textContentType = .fullStreetAddress
```

Try to use these solutions first.
# Passwords and security codes
```swift
let userTextField = UITextField()
userTextField.textContentType = .username

let passwordTextField = UITextField()
passwordTextField.textContentType = .password
```

Just tag fields with corresponding content types.

```swift
//security code
let securityCodeTextField = UITextField()
securityCodeTextField.textContentType = .oneTimeCode
```

```swift
//automatic strong password
let userTextField = UITextField()
userTextField.textContentType = .username

let newPasswordTextField = UITextField()
newPasswordTextField.textContentType = .newPassword //indicates autofill
```

[[Automatic STrong Passwords and Security Code AutoFill - 18]]

Big Sur now has security code autofill.  Also in AppKit.
`NSTextContentType`.
Also supports password manager apps.

Tag every text view in your app.  Address, email, phone numbe,r username, password, one-time code.

