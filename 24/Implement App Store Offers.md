

Learn how to engage customers with App Store Offers using App Store Connect, as well as the latest StoreKit features and APIs. Discover how you can set up win-back offers (a new way to re-engage previous subscribers) and generate offer codes for Mac apps. And find out how to test offers in sandbox and Xcode to make sure they work smoothly.

###  Present offer code redemption sheet on macOS - SwiftUI API - 4:25
```swift
// Present offer code redemption sheet on macOS - SwiftUI API

import SwiftUI
import StoreKit

struct MyView: View {

    @State var showOfferCodeRedemption: Bool = false

    var body: some View {
        Button("Redeem Code") {
            showOfferCodeRedemption = true
        }
        .offerCodeRedemption(isPresented: $showOfferCodeRedemption) { result in
            // Handle result
        }
    }
}
```

###  Choose preferred offer in a SubscriptionStoreView - 20:15
```swift
// Choose preferred offer in a SubscriptionStoreView

import SwiftUI
import StoreKit

struct MyView: View {
    let groupID: String
    
    var body: some View {
        SubscriptionStoreView(groupID: groupID)
            .preferredSubscriptionOffer { product, subscription, eligibleOffers in
                let freeTrialOffer = eligibleOffers
                    .filter { $0.paymentMode == .freeTrial }
                    .max { lhs, rhs in
                        lhs.period.value < rhs.period.value
                    }
                return freeTrialOffer ?? eligibleOffers.first
            }
    }
}
```

###  Check subscription entitlement and offer eligibility - 23:05
```swift
// Check subscription entitlement and offer eligibility

import StoreKit

func shouldShowMerchandising(for groupID: String, productIDs: [Product.ID]) async throws -> MerchandisingVisibility {
    // Get subscription status
    let statuses = try await Product.SubscriptionInfo.status(for: groupID)
    
    // Check if the customer is already entitled to the subscription
    let entitlement = SubscriptionEntitlement(for: statuses)
    if entitlement.autoRenewalEnabled {
        return .hidden
    }
    
    // Check for offers to show in merchandising UI
    let products = try await Product.products(for: productIDs)
    
    let isEligibleForIntroOffer = await Product.SubscriptionInfo.isEligibleForIntroOffer(for: groupID)
    if isEligibleForIntroOffer {
        let subscriptions = products.map {
            ($0, $0.subscription?.introductoryOffer)
        }
        return .visible(subscriptions)
    }
    
    // Check for eligible win-back offers
    let purchasedStatus = statuses.first { $0.transaction.unsafePayloadValue.ownershipType == .purchased }
    let renewalInfo = try purchasedStatus?.renewalInfo.payloadValue
    let bestWinBackOfferID = renewalInfo?.eligibleWinBackOfferIDs.first
    
    // Return the product with the offer if there is one
    if let bestWinBackOfferID {
        let subscriptions: [(Product, Product.SubscriptionOffer?)] = products.map {
            let winBackOffer = $0.subscription?.winBackOffers.first {
                $0.id == bestWinBackOfferID
            }
            return ($0, winBackOffer)
        }
        return .visible(subscriptions)
    }
    
    // Only return the product if there is no offer
    return .visible(products.map { ($0, nil) })
}

struct SubscriptionEntitlement {
    let isEntitled: Bool
    let autoRenewalEnabled: Bool
    
    init(for statuses: [Product.SubscriptionInfo.Status]) {
        let entitledStatuses = statuses.filter {
            $0.state == .subscribed || $0.state == .inBillingRetryPeriod || $0.state == .inGracePeriod
        }
        isEntitled = !entitledStatuses.isEmpty
        autoRenewalEnabled = entitledStatuses.contains { $0.renewalInfo.unsafePayloadValue.willAutoRenew }
    }
}

enum MerchandisingVisibility {
    case hidden
    case visible([(Product, Product.SubscriptionOffer?)])
}
```

###  Add a win-back offer to a purchase - 25:26
```swift
// Add a win-back offer to a purchase

import StoreKit

func purchase(_ product: Product, with offer: Product.SubscriptionOffer?) async throws {
    // Prepare the purchase options
    var purchaseOptions: Set<Product.PurchaseOption> = []
    
    // Add win-back offer to the purchase
    if let offer, offer.type == .winBack {
        purchaseOptions.insert(.winBackOffer(offer))
    }
    
    // Make the purchase
    try await product.purchase(options: purchaseOptions)
}
```

# Resources
* [Forum: App Store Distribution & Marketing](https://developer.apple.com/forums/topics/app-store-distribution-and-marketing?cid=vf-a-0010)
* [Generating a signature for promotional offers](https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase/subscriptions_and_offers/generating_a_signature_for_promotional_offers)
* [Message](https://developer.apple.com/documentation/storekit/message)
* [offer](https://developer.apple.com/documentation/storekit/transaction/4307076-offer)
* [PurchaseIntent](https://developer.apple.com/documentation/storekit/purchaseintent)
* [Setting up StoreKit Testing in Xcode](https://developer.apple.com/documentation/Xcode/setting-up-storekit-testing-in-xcode)
* [StoreKit views](https://developer.apple.com/documentation/storekit/in-app_purchase/storekit_views)
* [Submit feedback](http://feedbackassistant.apple.com)
* [Supporting subscription offer codes in your app](https://developer.apple.com/documentation/storekit/appstore/supporting_subscription_offer_codes_in_your_app)
* [Testing win-back offers in Xcode](https://developer.apple.com/documentation/storekit/in-app_purchase/testing_win-back_offers_in_xcode)