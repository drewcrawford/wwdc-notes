What's new in app clips.  Small part of an app, light, fast, etc.  Get what you need right when you need it.  messages, maps, spotlight, safari, out in the world with QR codes, etc.

ex: toast.  Scan a QR code to pay for your food from your table.  Opens from existing QR code printed on receipt.  Users check out faster, and installing the full app.  Building app clip into existing flow is a great way to streamline

# New size limit
Now up to 15MB in size.  Uncompressed, thinned app clip bundle
Set minimum version to iOS 16

Speed is key to a great experience.  [[Build light and fast App Clips]]

# Diagnostic tools
On iPhone/iPad, go to developer settings and select diagnostics.  Now, enter your URL.  Turn on developer settings by connecting your devicet ox code.

iOS checks your configuration.  If everything is ok, you'll see green checkboxes.  Otherwise, you'll see errors.  Fix problems liek banner not showing, webpage, etc.

* Physical code invocation
* Safari banner and iMessage availability
* Associated domains
# CloudKit
New in iOS 16, app clips can read data and resources stored in an icloud public database.  Share same data, code, in your app and app clip.  

Note that app clips can't access private database or shared database.  Or cloud documents, or key value store.  Because, when an app clip isn't in use, iOS will delete its data.

```swift
// Read your CloudKit public database from your App Clip

let container = CKContainer.default()
let publicDatabase = container.publicCloudDatabase
let record = try await publicDatabase.record(for:
    CKRecord.ID(recordName: "A928D582-9BB6-E9C5-7881-E4EAF615E0CD"))

if let title = record["Title"] as? String,
    let description = record["Description"] as? String {
        print(“Fetched a food item from CloudKit: \(title) \(description)")
}
```
# Keychain migration
When you want to transfer sensitive info, to your full app.  Your app wuold store this in shared container.  iOS saves the data when user upgrades.  Your full app reads the container, and stores info in the keychain.

However, the keychain is the ideal place to securely store this information.  New this year, when a user installs your app, items stored in the app clip's keychain are transferred.  Just store it in keychain!

Shared keychain groups and iCloud keychain are not supported.  Keeps promise to users that they won't stick around when the app clip isn't in use.
```swift
// Write authentication token to App Clip keychain

let addSecretsQuery: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecValueData as String: "smoothie-secret".data(using: .utf8),
    kSecAttrLabel as String: "foodsample-appclip"
]
SecItemAdd(addSecretsQuery as CFDictionary, nil)

// Read authentication token from app or App Clip

var readSecretsQuery: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecReturnAttributes as String: true,
    kSecAttrLabel as String: "foodsample-appclip",
    kSecReturnData as String: true
]
var secretsCopy: AnyObject?
SecItemCopyMatching(readSecretsQuery as CFDictionary, &secretsCopy)
```

# App clip experiences API
As your appclip grows,y ou'll have more and more experiences.  Each with own information and location.  Need to add/update all these experiences.  ASC web API automates this workflow.

Get App Clip resource ID
upload header image
Create advanced app clip experience

```swift
# Get the App Clip resource ID

GET /v1/apps/1234567890/appClips?filter[bundleId]=com.example.foodsample.Clip

# Response

{
    "data": {
        "attributes": {
            "bundleId": "com.example.foodsample.Clip"
        },
        "id": "726ad1bb-3e1b-40eb-a986-d8a9897e4f1e"
    }
}
```

```json
# Upload a header image for the advanced App Clip experience

POST /v1/appClipAdvancedExperienceImages
{
    "data": {
        "type": "appClipAdvancedExperienceImages",
        "attributes": {
            "fileName": "Hero_image_US.png",
            "fileSize": 43500
        }
    }
}

# Response

{
    "data": {
        "attributes": "..."
        "id": "91c52741-832b-48a2-8935-1797655edbe7"
    }
}
```
attributes, relationships, included.
attributes: information like action name, link, etc.  If tied to a maps location, add a place dictionary.  Add matching business place, action, coordinates.
Relationships: add appclip sresource id and header id
included: localized title and subttitle.
```json
# Create advanced App Clip experience

POST /v1/appClipAdvancedExperiences
{
    "data": {
        "type": "appClipAdvancedExperiences",
        "attributes": {
            "action": “OPEN",
            "businessCategory": "FOOD_AND_DRINK",
            "defaultLanguage": "EN",
            "isPoweredBy": true,
            "link": "https://fruta.example.com/restauraunt/simply_salad",
            "place": {
                "names": [ "Caffe Macs" ],
                "mapAction": "RESTAURANT_ORDER_FOOD",
                "displayPoint": {
                    "coordinates": { "latitude": 37.33611, "longitude": -122.00731 },
                    "source": "CALCULATED"
                }
            }
        },
        "relationships": {
            "appClip": {
                "data": {
                    "type": "appClip",
                    "id": "726ad1bb-3e1b-40eb-a986-d8a9897e4f1e"
                }
            },
            "headerImage": {
                "data": {
                    "type": "appClipAdvancedExperienceImages",
                    "id": "91c52741-832b-48a2-8935-1797655edbe7"
                }
            }
        },
        "included": {
            "type": "appClipAdvancedExperienceLocalizations",
            "attributes": {
                "language": "EN",
                "subtitle": "Fresh salad every day",
                "title": "Simply Salad"
            }
        }
    }
}
```

[[Automating App Store Connect - 18]]
[[What's new in appstore connect]]

# Wrap up
* 15mb size limit
* diagnostics tools
* CloudKit and keychain
* app clip experiences API
[[21/What's new in App Clips]]
[[Design Great App Clips]]

* https://developer.apple.com/forums/tags/wwdc2022-10097
* https://developer.apple.com/forums/create/question?&tag1=295&tag2=425030
* https://developer.apple.com/documentation/app_clips
