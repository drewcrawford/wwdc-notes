Learn how to write clear and succinct purpose strings to help people understand why your app needs access to protected resources like their camera, location, and health data. We'll take you through best practices to help craft concise purpose strings and show you how you can improve wording in permission requests.

Guideline 5.1.1 details essential privacy requirements.

# Background on permission requests
Identity.  

Apps must request user's permission.  They decide which apps have access to what.  People need to know more about why your app needs access.

Short message you provide appearing on every permission request.  Why you need access, what you will do with the data.  Define these messages on an app's info plist by setting a string value to a resource-specific key.

coffee cup phone number example.

# Purpose strings in review

They should
* explain specifically how the app will use the data
* provide an example of how the data will be used.

# Writing purpose strings

not "To use location."

Be sure to localize purpose strings.  We recommend testing your app in different localizations.

ex: Social app - tracking request.  They need ATT.  "Your data will be used to deliver personalized advertising for products that are more relevant to you."  Or "you will still see advertisements if you decline tracking, but they may be less relevant to you."

many apps integrate 3rd-party SDKs.  Various features, services.  They can also reference APIs that may try to access protected resources.  If your app has a reason to access these resources.  If not, remove reference to these APIs from app's binary.

You're responsible for 3rd party code.  Be sure to familiarize yourself with any protected resources they may try to access.  

Is there such a thing as too much info?


# Resources
* https://developer.apple.com/app-store/review/guidelines/#data-collection-and-storage
* https://developer.apple.com/design/human-interface-guidelines/patterns/accessing-private-data
* https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/requesting_access_to_protected_resources
