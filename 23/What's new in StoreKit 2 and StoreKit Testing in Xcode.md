#storekit 

Get to know the latest enhancements to StoreKit 2 and StoreKit Testing in Xcode. Discover API updates for promoted in-app purchases, StoreKit messages, the Transaction model, the RenewalInfo model, and the App Store sheet for managing subscriptions. Learn how to upgrade to SHA-256 for on-device receipt validation and use APIs to create SwiftUI views. We'll also help you get started with StoreKit Testing in Xcode so that you can debug and test your in-app purchases and subscriptions. Meet the Transaction Inspector, explore the latest updates to the StoreKit configuration editor, and find out how you can simulate StoreKit errors to test your app's error handling.

# API updates
promoting in-app purchases
* display in-app purchase in app store
* appear in app store search results
* promotional image
* higher product visibility
* easy setup in ASC

[[23/What's new in AppStore Connect|What's new in AppStore Connect]]



## Create a listener for promoted in-app purchases - 1:42
```swift
// Create a listener for promoted in-app purchases
import StoreKit

let promotedPurchasesListener = Task {
    for await promotion in PurchaseIntent.intents {
        // Process promotion
        let product = promotion.product

        // Purchase promoted product
        do {
            let result = try await product.purchase()
            // Process result
        }
        catch {
            // Handle error
        }
    }
}
```

customize promotions as displayed in app store, for example reorder or hide existing purchases.

## Check promotion order - 2:57
```swift
// Check promotion order
import StoreKit

do {
    let promotions = try await Product.PromotionInfo.currentOrder

    if promotions.isEmpty {
        // No local promotion order set
    }

    for promotion in promotions {
        let productID = promotion.productID
        let productVisibility = promotion.visibility
        // Check promoted products
    }
}
catch {
    // Handle error
}
```

## Set a promotion order - 3:26
```swift
// Set a promotion order
import StoreKit

let newPromotionOrder: [String] = [
    "acorns.individual",
    "nectar.cup",
    "sunflowerseeds.pile"
]

do {
    try await Product.PromotionInfo.updateProductOrder(byID: newPromotionOrder)
}
catch {
    // Handle error
}
```

## Update promotion visibility - 4:02
```swift
// Update promotion visibility
import StoreKit

// Hide “acorns.individual”
do {
    try await Product.PromotionInfo.updateProductVisibility(.hidden, for: "acorns.individual")
}
catch {
    // Handle error
}
```

## Update promotion visibility (alternative method) - 4:17
```swift
// Update promotion visibility
import StoreKit

do {
  let promotions = try await Product.PromotionInfo.currentOrder

  // Hide the first product
  if var firstPromotion = promotions.first {
    firstPromotion.visibility = .hidden
    try await firstPromotion.update()
  }
}
catch {
  // Handle error
}
```

need to call update here.

[[Meet StoreKit 2]]
data model changes.

transaction -> storefront, storefrontCountryCode, reason

renewalinfo -> renewalDate: when subscription will be processed

availability xcode 15
most of them work retroactively

StoreKit2 updates
* receive and display app store messages
* optioanlly defer or suppress messages
* [[What's new with in-app purchases]]

choose wehether to defer or suppress message.  this has stuff like billing problems

New billingIssue message reason
resolve billing issues without leaving your app
available to test in sandbox
enabled for all customers later this year

[[Explore in-app purchase testing]]

## app receipt signing updates
from sha1 to sha256.  Handle this one properly

testable in sandbox on june 20
new and updated apps on aug 14
see developer website




# Build SwiftUI apps

new apis to create swiftui views for single-purchases, etc.

## Product view - 8:32
```swift
// Product view
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
    let productID: String
    let productImage: String

    var body: some View {
        ProductView(id: productID) {
            BirdFoodProductIcon(for: productID)
        }
        .productViewStyle(.large)
    }
}
```

## Store view - 8:52
```swift
// Store view
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
    let productIDs: [String]

    var body: some View {
        StoreView(ids: productIDs) { product in
            BirdFoodIcon(productID: product.id)
        }
    }
}
```

## Subscription view - 9:19
```swift
// Subscription view
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
    let groupID: String

    var body: some View {
        SubscriptionStoreView(groupID: groupID)
    }
}
```

Many possible customizations.  For example, check out how changing  a few lines of code can alter the storeview.

we also have a managed subscription sheet.

can also jump right into the group you want to show.

(code sample not available)

[[Meet StoreKit for SwiftUI]]



# StoreKit Testing in Xcode

supported on simulator and device.

Transaction manager -> new functionality for debugging and testing your apps.  In navigator can see connected devices and simulator, using storekit configuration for testing.

Debug->StoreKit->Manage transactions.  here is the transaction manager view.
indicator dot shows the app actively being debugged.

I have the opportunity to configure this purchase.  Default options are also valid.  Add a transaction in transaction manager.

subscription options.  I can pick an offer code.  Customers have to type in offer codes, but to make testing easier, we provide a list.

PUrchase date -> can backdate.
renew automatically or autorenew.  

ios 17 - no debug session required, customizable purchases
ios 15 - requires debug session, no purchases available

configuration settings - purchase options, etc.
In prior versions of xcode, you could find these in the editor menu.  Still there, no we surface them in UI instead.

quick/reliable testing.  You can find these new configurable rates.  New failure types.  Pick an error that the API should throw.  Numerous APIs are supported.  These take effect immediately.


## Simulated off-device purchase using StoreKitTest - 21:09
```swift
// Simulated off-device purchase using StoreKitTest
import StoreKit
import StoreKitTest

func testSubscriptionRenewal() async throws {
    let session = try SKTestSession(configurationFileNamed: "Store")

    let oneYearInterval: TimeInterval = (365 * 24 * 60 * 60)
    let transaction = try await session.buyProduct(
        identifier: "birdpass.individual",
        options: [
            .purchaseDate(Date.now - oneYearInterval)
        ]
    )

    // Inspect transaction
}
```

purchase options are only supported for testing.  Can programmatically do this like from the transaction manager I guess.

Can also do errors when loading products, programmatic errors.  `setSimulatedSerror` API.

## Set a simulated purchase error when loading products - 21:48
```swift
// Set a simulated purchase error when loading products
import StoreKit
import StoreKitTest

func testLoadProducts() async throws {
    let session = try SKTestSession(configurationFileNamed: "Store")
    let productIDs = [
        "acorns.individual",
        "nectar.cup"
    ]

    // Set a simulated error, then load products, expecting an error
    session.setSimulatedError(.generic(.networkError), forAPI: .loadProducts)
    do {
        _ = try await Product.products(for: productIDs)
        XCTFail("Expected a network error")
    }
    catch StoreKitError.networkError(_) {
        // Expected error thrown, continue...
    }
    // Disable simulated error
    session.setSimulatedError(nil, forAPI: .loadProducts)
}
```

APIs for new subscription renewal rates.

## Set a faster subscription renewal rate in a test session - 22:24
```swift
// Set a faster subscription renewal rate in a test session
import StoreKit
import StoreKitTest

func testSubscriptionRenewal() async throws {
    let session = try SKTestSession(configurationFileNamed: "Store")

    // Set renewals to expire every minute
    session.timeRate = .oneRenewalEveryMinute

    let transaction = try await session.buyProduct(identifier: "birdpass.individual")

    // Wait for renewals and inspect transactions
}
```

# Wrap up
* adopt sk2
* try new sk2 apis
* build apps 8using the in-app merchandising views
* file bugs we won't read

[[Meet StoreKit 2]]
[[What's new in StoreKit testing]]
[[Explore in-app purchase testing]]
[[What's new with in-app purchases]]

https://developer.apple.com/documentation/storekit/in-app_purchase/testing_in-app_purchases_with_sandbox/testing_failing_subscription_renewals_and_in-app_purchases
https://developer.apple.com/documentation/storekit/message
https://developer.apple.com/documentation/storekit/in-app_purchase/supporting_promoted_in-app_purchases_in_your_app
https://help.apple.com/app-store-connect/#/dev45b03fab9
https://developer.apple.com/documentation/Xcode/setting-up-storekit-testing-in-xcode
https://developer.apple.com/documentation/storekit

