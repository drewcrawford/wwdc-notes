
We believe privacy is a fundamental human right.  In addition to data techniques, we want to make sure people have transparency and control.

Apps we build are integral parts of people's lives.  Privacy nutrition labels help people learn more.

# Privacy manifests
We've heard that it can be challenging to hear about what 3rd party sdks are doing.  

can include a privacy manifest in their sdk.  

PrivacyInfo.xcprivacy.

see docs: App privacy details on the app store
definitions are for the same as the nutrition labels.


# Privacy report
brings all this information to one place.
when building your app, right click the archive and choose 'generate privacy report'.

privacy report is a pdf and easy to use.  It is organized in a similar way to the nutrition label.  Easily reference this report when providing privacy details in ASC.

[[Create your Privacy Nutrition Label]]

# Tracking domains
Understand and control network connections from your app.

many 3rd party sdks check status before tracking people.  But maybe you have to manually disable tracking?  This could lead to edge cases.  To make it easier for you and 3rd party developers to avoid tracking
* declare in privacy manifest
* blocked until user allows tracking

you can also do this for your app.  Preserves your intention to not track users without therir permission.  
see docs: user privacy and data use
[[Explore App Tracking Transparency]]

In some cases, you might use domains for both purposes.  An approach that you can take is
* separate tracking and non-tracking functionality into different domains

check behavior in instruments.  We show these as POIs.

Note that fingerprinting is **never** allowed.  Regardless of whether the user gives you permission to track.
# Required reason APIs

To support important usecases, while avoiding fingerprinting, there's a new category of required reason APIs.
Platform APIS with high potential for misuse.
approved reasons for access
NSFileSystemFreeSize (disk space).
* check whether there is sufficient disk space before writing files to disk
* Listed in developer documentation

Total number of required reason APIs is small.  But you might use one or more.  If you have a usecase for an API category that is not covered by an approved reason.  The docs link to a feedback form.

Apps and SDKs
* may only access for approved reasons
* must declare reasons in respective privacy manifest.

Just like nutrition labels, privacy manifests help you make decisions about your dependencies.  Better picture of your app's privacy story.

* identified third-party SDKs
* high impact on user privacy
* listed in developer documentation
* "Privacy-impacting SDKs"
To be included in apps:
* must have privacy manifest
* xcode also supports SDK signatures
* apps that include a privacy-impacting SDK must ensure the SDK is signed.
[[Verify app dependencies with digital signatures]]

## new and updated apps
* manifests, signatures, required reasons
* informational emails starting fall 2023
* required reason APIs without approved reasons
* expected starting spring 2024 - part of app review

# next steps
app developers:
* ask for SDK privacy manifests
* refer to xcode privacy report
SDK developers:
* adopt signatures and manifests
declare tracking domains and required reason API usage


# Resources
* https://developer.apple.com/app-store/app-privacy-details
* https://developer.apple.com/app-store/user-privacy-and-data-use/
