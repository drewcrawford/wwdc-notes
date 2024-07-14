Learn how to build and deliver even better purchase experiences using the App Store In-App Purchase system. We'll demo new StoreKit views control styles and new APIs to improve your subscription customization, discuss new fields for transaction-level information, and explore new testability in Xcode. We'll also review an important StoreKit deprecation.

# Core api enhancements

Historically, we included transactions for various transaction types.  So when they purchase a consumable, finished consumable was not included.  Now isntead of having to manually track finished consumables, we track everything regardless of finish state.

Opt-in behavior
Enabled through your app's info plist file.  SKIncludeConsumableInAppPurchaseHistory
Also available from app store server api


Transaction type - currency.  As name suggests, it tells you the currency used.  `price` -> price you configured in appstore connect.  

renewalInfo -> currency, renewalPrice.  Important o note that the renewal price should not be interpreted without currency.  Access when building via xcode 16.  Even available on iOS 15.

## win-back offers
Re-engaged churned subscribers.  Powerful new tools that allow you to customize eligibliity rules
storekit message automatic merchandising
Promoted on the app store
[[Implement App Store Offers]]

# merchandise using swiftui
What better ways to showcase new features besides destination video.

For more on storekit configuration file see [[What's new in StoreKit testing]]
[[Introducing StoreKit testing in Xcode]]

* premium plan
* basic plan
* whether it renews monthly or yearly

Declare a subscription store view and provide groupID for subscription.  With just one line of code, off to a great start.  Structure my store so it's clearly visible to customers.  

Define an option group using a condition representing which objects are included.  Include products whenever a level of service is premium.  Create a n neum earlier called StreamingPassLevel.  Notice how store updated for premium plans.

### 4:26 - Destination Video Shop
```swift
import StoreKit
import SwiftUI

struct DestinationVideoShop: View {
    var body: some View {
        SubscriptionStoreView(groupID: Self.subscriptionGroupID) {
            SubscriptionOptionGroupSet { product in
                StreamingPassLevel(product)
            } label: { streamingPassLevel in
                Text(streamingPassLevel.localizedTitle)
            } marketingContent: { streamingPassLevel in
                StreamingPassMarketingContent(level: streamingPassLevel)
                StreamingPassFeatures(level: streamingPassLevel)
            }
        }
        .subscriptionStoreControlStyle(.compactPicker, placement: .bottomBar)
    }
}
```

Two groups -> tab view.  New layout makes my subscription choices easier to understand.

Store view allows a swiftui view as custom marketing content.  

Using subscription option groups is great.  Provide marketing content view depending on active group.  Explain the value of the basic plan.  When I change the active tab, the marketing content explains the basic plans.

Streaming Pass + operates a lot of value to customers, etc.  We hav ea lot of text.  But now I can use compact picker style.  Now the subscription option picker takes up less space than before.

* subscription option group
* groupset

Just showed you how to use option grups, etc.
tab style - clear distinctions among subscription plan options.  Tabs is one of the styles available for presenting subscription option groups.  Defines how storekit defines your groups.


### 9:06 - Subscription Option Groups - Tabs style
```swift
SubscriptionStoreView(groupID: Self.subscriptionGroupID) {
    SubscriptionOptionGroupSet { product in
        StreamingPassLevel(product)
    } label: { streamingPassLevel in
        Text(streamingPassLevel.localizedTitle)
    } marketingContent: { _ in
        StreamingPassMarketingContent()
    }
}
.subscriptionStoreControlStyle(.compactPicker, placement: .bottomBar)
.subscriptionStoreOptionGroupStyle(.tabs)
```
### 9:20 - Subscription Option Groups - Links style
```swift
SubscriptionStoreView(groupID: Self.subscriptionGroupID) {
    SubscriptionOptionGroupSet { product in
        StreamingPassLevel(product)
    } label: { streamingPassLevel in
        Text(streamingPassLevel.localizedTitle)
    } marketingContent: { _ in
        StreamingPassMarketingContent()
    }
}
.subscriptionStoreControlStyle(.compactPicker, placement: .bottomBar)
.subscriptionStoreOptionGroupStyle(.links)
```

OPtion groups
* organize products into hierarchies
* groups within groups
* SubscriptionOptionSection
* subscriptionOptionGroupSet
* SubscriptionPeriodGroupSet
Place controls in a subscription store
Not all placements are available in every control style.
Statically validate placements


Platform-specific placements in iOS 18 and aligned releases.  

On tvOS, butotn style is only standard control style.  But it has a number of placements available including automatic, leading, trailing, bottom

leading: tvOS can now lay out horizontally
trailing: new default
bottom: vertical layout

storekit views
new standard control styles
* compact picker - best used with 2/3 plan options
* paged picker - 2/3 plans in a row.
* Paged prominent picker - paged picker style, but adds a prominent border/scale effect.
New styles are great for reducing usage of vertical space.  

Total number of standard styles from 3 to 6.  Essential elements of a subscription store.  Create best-in-class IAP experiences.  To further customize appearance, starting in iOS 18 you can create your own custom control styles.  We're amking the same primitive sSK uses to create these styles.

Conform to `SubscriptionStoreControlStyle`.  


### 13:41 - Custom control style implementation
```swift
import StoreKit
import SwiftUI

struct BadgedPickerControlStyle: SubscriptionStoreControlStyle {
    func makeBody(configuration: Configuration) -> some View {
        SubscriptionPicker(configuration) { pickerOption in
            HStack(alignment: .top) {
                VStack(alignment: .leading) {
                    Text(pickerOption.displayName)
                        .font(title2.bold())
                    Text(priceDisplay(for: pickerOption))
                    if pickerOption.isFamilyShareable {
                        FamilyShareableBadge()
                    }
                    Text(pickerOption.description)
                }
                Spacer()
                SelectionIndicator(pickerOption.isSelected)
            }
        } confirmation: { option in
            SubscribeButton(option)
        }
    }
}

struct DestinationVideoShop: View {
    var body: some View {
        SubscriptionStoreView(groupID: Self.subscriptionGroupID) {
            SubscriptionPeriodGroupSet { _ in
                StreamingPassMarketingContent()
            }
        }
        .subscriptionStoreControlStyle(BadgedPickerControlStyle())
    }
}
```




# testing in xcode

Ensure your IAP is the best it can be.  

## StoreKit configuration

New in xc16, you can test your privacy policy locally.  App policies section.  Open an editor for EULA/privacy policy.  Values are displayed on your subscription store view.

Test localizations for your subscription group's display name.  
Plus button opens an editor where you can add your localized group's display name.

Winback offer.  Similar to other types you're familiar with.  [[Implement App Store Offers]]

Test IAP image from your testing configuration.  Since image is for local testing purposes only, add any image you want.  

Use ProductView/Storeview and set `prefersPromotionalIcon`.  See [[Meet StoreKit for SwiftUI]]

Dialogs.  Can disable system dialogs to choose the default option when a dialog will be presented such as during an IAP.  Useful for UI automation tests and you only want to test the default flows.

## transaction manager

Testing and debugging IAPs.  Inspect transactions, simulate purchases for all apps, etc.  Works across all devices and simulators.  

Send purchase intents directly from xcode.  

Purchase intent - customer, app store, etc.
Purchase data is sent to your app, which you can use to complete the purchase.

[[What's new in StoreKit 2 and StoreKit Testing in Xcode]]

I've added code to listen for purchases.  Transaction manager.  New control to decide between regular purchase and purchase intent.  

If you choose to handle purchase intents, can create your own UI to merchandise the product.  Payment sheet will present itself, etc.

Can now test billing issue messages directly in app.  When your subscription cannot renew due to a billing problem.  SK will prompt you to resolve the issue.  Test via transaction manager.

On iOS 18, you get a sheet in the app.  Enable 'billing retry' options in configuration setting.  Badge appears in transaction manager to simulate billing retry.  

# Update to storekit2

If you still use original API, beginning with iOS 18, original API for IAP is deprecated, including unified receipt.  Your existing apps will work, but we're not introducing new enhancements.  We strongly recommend updating your existing app to use SK2.  

* Swift api for transactions
* swift api for subscription renewal information
* automatic cryptographic validation
* async/await
* back deploys to iOS 15.

# Wrap up
* adopt storekit2
* Elevate your merchandising experience with StoreKit views
* Test within Xcode

[[Meet StoreKit 2]]
[[What's new in StoreKit testing]]
[[Meet StoreKit for SwiftUI]]



# Resources
* https://developer.apple.com/documentation/storekit/in-app_purchase
* https://developer.apple.com/storekit/
* https://developer.apple.com/documentation/storekit/message
* https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase
* https://developer.apple.com/documentation/storekit/product/subscriptioninfo/renewalinfo
* https://developer.apple.com/documentation/Xcode/setting-up-storekit-testing-in-xcode
* https://developer.apple.com/documentation/storekit/in-app_purchase/storekit_views
* https://developer.apple.com/documentation/Xcode/testing-in-app-purchases-with-storeKit-transaction-manager-in-code
* https://developer.apple.com/documentation/storekit/transaction/transaction_properties
