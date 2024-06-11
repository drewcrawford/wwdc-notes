Learn how to build and deliver even better purchase experiences using the App Store In-App Purchase system. We'll demo new StoreKit views control styles and new APIs to improve your subscription customization, discuss new fields for transaction-level information, and explore new testability in Xcode. We'll also review an important StoreKit deprecation.

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
