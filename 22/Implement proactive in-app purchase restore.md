Identify customers' new, current, past purchases.  SK2 and the original storekit.  Optimize your apps for all customers.

# Proactive IAP Restore
* use data on device
* identify current and past purchases automatically
* no customer action required
* Tailor customer experience to purchase history

Using SK to optimize your app's experience for new, existing, and past ucstomers on all devices etc.

For active subscribers on a new device, knowing which option to choose isn't always clear.  Proactive IAP Restore best practice.  Ideally, we restore automatically even on new devices.

SK2 on iOS 15 and newer.  If you suppport previous versions, I'll discuss that in SK1.

# Personalized onboarding
* non-consumables
* non-renewing subscriptions
* auto-renewable subscriptions

let's call them both "subscriptions".

* New customers => signed in apple ID, but no current/past transactions.  Default merchanidisng experience?
* Purchased /active subscriber.  Customer is obligated to grant the customer access to purchased product or service.  `originalTransactionId`.  Persists for the customer's appleid and storefront.  
	* Associate with an account on your system.  Can be anonymous account or one they created.  
	* Knowing this is critical when leveraging server notifications.  ex, failed to autorenew.
		* https://apps.apple.com/account/billing
* Inactive purchase / inactive subscriber.  Customers who previously made in-app purchases but are no longer entitled to that product/service due to expiring, etc.
	* for subscriptions, determined by expires date.
	* for all types, can have a revocation date.  ex, family sharing, refunds.
	* Consider presenting subscription offers to win them back.
		* billing retry state => https://apple.apple.com/account/billing

## Other considerations
* multiple product offerings
* Off-platform activity
* App Store Server Notifications V2

## App Store Server Notifications v2
* learn about migrating to v12
* v2 compatible with storekit2 and original storekit
* best practices

[[Explore in-app purchase integration and migration]]

# Implementation
Now let's walk through implementation details.  I'll use SK demo app.  Note that the demo app will bea vailable for download.

* cars
* navigation service => subscription
* non-renewing subscription

execute 3 steps.  Most important is that these steps are completed before a buy button is shown

1.  Listen for transactions
	2. at app launch, iterate through Transaction.updates
	3. Transactions can apepar unexpectedly.  Family sharing, code redemption, etc.
	4. Deliver unfinished transactions

need text here to fix obisidan bug https://forum.obsidian.md/t/after-ending-a-list-highlighted-code-blocks-are-broken/41079
```swift
//Transaction Listener at app launch
func listenForTransactions() -> Task<Void, Error> {
    return Task.detached {
        //Iterate through any transactions which didn't come from a direct call to `purchase()`.
        for await result in Transaction.updates {
            do {
                let transaction = try self.checkVerified(result)
                
                //Deliver products to the user.
                await self.updateCustomerProductStatus()
                
                //Always finish a transaction
                await transaction.finish()
                
            } catch {
                //StoreKit transaction failed verification, don't deliver content to user.
                print("Transaction failed verification")
            }
        }
    }
}
```

Once a transaction is finished, it will no longer be returned to your app.

2.  Identitfy customer product state
	3. identify active prucahses with `ucrrentEntitlements`
	4. For auto-renewal usbscriptions, use `Product.SubscriptionInfo.RenewalState`

workaround  https://forum.obsidian.md/t/after-ending-a-list-highlighted-code-blocks-are-broken/41079
```swift
//Determine customer product state
func updateCustomerProductStatus() async {
   
    var purchasedCars: [Product] = []
    var purchasedSubscriptions: [Product] = []
    var purchasedNonRenewableSubscriptions: [Product] = []

    //Iterate through all of the user's purchased products.
    for await result in Transaction.currentEntitlements {

       do {
         //First check if the transaction is verified. If the transaction is not verified
         //we'll catch the `failedVerification` error.
         let transaction = try checkVerified(result)

         //Check the `productType` of the transaction and get the corresponding product 
           from the store.
         switch transaction.productType {

         case .nonConsumable:
                    
             if let car = cars.first(where: { $0.id == transaction.productID }) {
                        purchasedCars.append(car)
             }
         //..
```


```swift
//Determine customer product state

case .nonRenewable:
 
    if let nonRenewable = nonRenewables.first(where: { $0.id == transaction.productID }),
          transaction.productID == "nonRenewing.standard" {

 //Non-renewing subscriptions have no inherent expiration. 
         
     let currentDate = Date()
     let expirationDate = Calendar(identifier: .gregorian).date(byAdding:
                                                    DateComponents(year: 1),
                                                    to: transaction.purchaseDate)!

    if currentDate < expirationDate {
        purchasedNonRenewableSubscriptions.append(nonRenewable)

      }
   }
//..
```

Added additional logic to add expiration date for non-renewing subscription.

```swift
//Determine customer product state

case .autoRenewable:
  
  if let subscription = subscriptions.first(where: { $0.id == transaction.productID }) {
purchasedSubscriptions.append(subscription) }
     default:
       break
      }
  } catch {
      print()
  }
}
//Update the Store information with the purchased products.
self.purchasedCars = purchasedCars
self.purchasedNonRenewableSubscriptions = purchasedNonRenewableSubscriptions
self.purchasedSubscriptions = purchasedSubscriptions

//Check subscriptionGroupStatus to learn auto-renewable subscription state
subscriptionGroupStatus = try? await subscriptions.first?.subscription?.status.first?.state

}
```

to account for billing retry, etc.  Our `subscriptionGruopsTatus` uses the API Product.SubscriptionInfo.RenewalState

3.  Personalize onboarding experience

```swift
//Updating my car view at app launch

if store.purchasedCars.isEmpty && store.purchasedNonRenewableSubscriptions.isEmpty 
                               && store.purchasedSubscriptions.isEmpty {
        
               VStack {
                    Text("SK Demo App")
                        .bold()
                        .font(.system(size: 50))
                        .padding(.bottom, 20)
                    Text("🏎💨")
                        .font(.system(size: 120))
                        .padding(.bottom, 20)
                    Text("Head over to the shop to get started!")
                        .font(.headline)
                    NavigationLink {
                        StoreView()
                    }
            //…

     }
   }
}
```

for active purchases:

```swift
//Updating my car view at app launch

else {
    List {
        Section("My Cars") {
          
        if !store.purchasedCars.isEmpty {
             
              ForEach(store.purchasedCars) { product in
                  NavigationLink {
                      ProductDetailView(product: product)
                 } label: {
                      ListCellView(product: product, purchasingEnabled: false)
                }
           } 
               } else {

          Text("You don't own any car products. \nHead over to the shop to get started!")

      }
   }
//…
```

If customer has navigation service.

```swift
//Updating my car view at app launch

Section("Navigation Service") {
   
     if !store.purchasedNonRenewableSubscriptions.isEmpty || 
         !store.purchasedSubscriptions.isEmpty {

          ForEach(store.purchasedNonRenewableSubscriptions) { product in
               NavigationLink {
                  ProductDetailView(product: product)
              } label: {
                  ListCellView(product: product, purchasingEnabled: false)
              }
          }

          ForEach(store.purchasedSubscriptions) { product in
               NavigationLink {
                   ProductDetailView(product: product)
               } label: {
                   ListCellView(product: product, purchasingEnabled: false)
              }
        }
    }
```

Inactive subscribers.  Expuired, revoked, billing retry state.
```swift
//Updating my car view at app launch

else  {
       
  if let subscriptionGroupStatus = store.subscriptionGroupStatus {
                              
     if subscriptionGroupStatus == .expired || subscriptionGroupStatus == .revoked {
                                   
         Text("Welcome Back! \nHead over to the shop to get started!")
                   
     } else if subscriptionGroupStatus == .inBillingRetryPeriod {
                                   
        //Provide a deep link from your app to https://apps.apple.com/account/billing.
         Text("Please verify your billing details.")
                               
     }
     } else {

       Text("You don't own any subscriptions. \nHead over to the shop to get started!")
     }
   }         
}
```

Our demo app proactively restored purchases.  Your app can use APIs and data regularly available.  So we covered this with SK2.

## SK1
* supports proactive restore on iOS7+
* Use app store `verifyReceipt` endpoint
* App receipt always present on device
	* on app store
	* but for sandbox etc, only after purchase.  Can consider this like a new customer.
	* Include password for latest transactions

Difference here is in step 2.

2. Indetify customer product state
	3. retrieve app receipt with `appstoreReceiptURL`
	4. Send app receipt to your server, validate with `verifyReceipt` endpoint
	5. Determine customer product state

```swift
//Fetch App Receipt Data

public func getReceipt() {

    if let appStoreReceiptURL = Bundle.main.appStoreReceiptURL,
        FileManager.default.fileExists(atPath: appStoreReceiptURL.path) {

        do {
            let receiptData = try Data(contentsOf: appStoreReceiptURL, 
                                          options: .alwaysMapped)
            print(receiptData)
            
            let receiptString = receiptData.base64EncodedString(options: [])
            
            print("receipt send it to your server: \(receiptString)")

            // Read receiptData
        }
        catch { 
            print("Couldn't read receipt data with error: " + error.localizedDescription) 
        }
    }
}
```

To determine customer product state, we used the engine fro WWDC 2020.
we enhanced it to
* supprot new customer product state
* support non-consumables and non-renewing subscriptions product types
see [[Architecting for Subscriptions]]

# Best practices
* restore purchases button anyway
* CloudKit
* Testing
	* sandbox, testflight, and xcode storekit testing
	* app store receipt not always present in testing
	* always present when installed from the appstore

# Wrap up
* proactively check for purchases
* Tailor app experience on launch
* Stay up to date with app store server notifications v2

[[What's new with in-app purchases]]


* https://developer.apple.com/forums/tags/wwdc2022-110404
* https://developer.apple.com/forums/create/question?&tag1=129&tag2=571030
* https://developer.apple.com/documentation/storekit/in-app_purchase/implementing_a_store_in_your_app_using_the_storekit_api
* https://developer.apple.com/storekit/
* https://developer.apple.com/documentation/appstoreservernotifications
* https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase/subscriptions_and_offers/determining_service_entitlement_on_the_server
* https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase/subscriptions_and_offers/reducing_involuntary_subscriber_churn
* https://developer.apple.com/documentation/cloudkit
