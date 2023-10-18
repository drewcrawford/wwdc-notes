#swiftui #storekit 
Discover how you can use App Store product metadata and Xcode Previews to add in-app purchases to your app with just a few lines of code. Explore a new collection of UI components in StoreKit and learn how you can easily merchandise your products, present subscriptions in a way that helps users make informed decisions, and more.

# Merchandising IAP
get product data, customer status.
combine this into human interface.

woudln't it be nice if we could abstract all the stuff into a view.  I'm excited to introduce new storekit APIs for merchandising.

StoreKit + SwiftUI
* StoreView
* productView
* SubscriptionStoreView

iPad, iPhone, mac, apple watch, apple TV.

sample app: backyard birds.
[[What's new in StoreKit testing]]
[[Introducing StoreKit testing in Xcode]]

example app

```swift
import SwiftUI

struct BirdFoodShop: View {
 
  var body: some View {
    Text("Hello, world!") 
  }
  
}
```

query our product IDs.
Now we have a functioning merchandising view.
```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
  @Query var birdFood: [BirdFood]
 
  var body: some View {
    StoreView(ids: birdFood.productIDs) 
  }
  
}
```

names, descriptions, etc., directly from the appstore / ASC.

More subtle but important considerations such as caching, IAP disable, etc.

Use a viewbuilder per cell?

```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
  @Query var birdFood: [BirdFood]
	
  var body: some View {
    StoreView(ids: birdFood.productIDs) { product in 
      BirdFoodProductIcon(productID: product.id)
    }
  }
 
}
```

Our product automatically adjusts to different platforms, etc.

```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
  @Query var birdFood: [BirdFood]
 
  var body: some View {
    ScrollView {
      VStack(spacing: 10) {
        if let (birdFood, product) = birdFood.bestValue {
          
        }
      }
      .scrollClipDisabled()
    }
    .contentMargins(.horizontal, 20, for: .scrollContent)
    .scrollIndicators(.hidden)
    .frame(maxWidth: .infinity)
    .background(.background.secondary)  
  }
  
}
```

we can add a decorative icon for this, just like the trailing closure.



```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
  @Query var birdFood: [BirdFood]
 
  var body: some View {
    ScrollView {
      VStack(spacing: 10) {
        if let (birdFood, product) = birdFood.bestValue {
          ProductView(id: product.id)
        }
      }
      .scrollClipDisabled()
    }
    .contentMargins(.horizontal, 20, for: .scrollContent)
    .scrollIndicators(.hidden)
    .frame(maxWidth: .infinity)
    .background(.background.secondary)  
  }
  
}
```

let's add a background.
```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
  @Query var birdFood: [BirdFood]
 
  var body: some View {
    ScrollView {
      VStack(spacing: 10) {
        if let (birdFood, product) = birdFood.bestValue {
          ProductView(id: product.id) {
            BirdFoodProductIcon(
              birdFood: birdFood,
              quantity: product.quantity
            )
          }
        }
      }
      .scrollClipDisabled()
    }
    .contentMargins(.horizontal, 20, for: .scrollContent)
    .scrollIndicators(.hidden)
    .frame(maxWidth: .infinity)
    .background(.background.secondary)  
  }
  
}
```

```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
  @Query var birdFood: [BirdFood]
 
  var body: some View {
    ScrollView {
      VStack(spacing: 10) {
        if let (birdFood, product) = birdFood.bestValue {
          ProductView(id: product.id) {
            BirdFoodProductIcon(
              birdFood: birdFood,
              quantity: product.quantity
            )
          }
          .padding()
          .background(.background.secondary, in: .rect(cornerRadius: 20))
        }
      }
      .scrollClipDisabled()
      Text("Other Bird Food")
        .font(.title3.weight(.medium))
        .frame(maxWidth: .infinity, alignment: .leading)
      ForEach(birdFood.premiumBirdFood) { birdFood in
        BirdFoodShopShelf(title: birdFood.name) {
          
        }
      }
    }
    .contentMargins(.horizontal, 20, for: .scrollContent)
    .scrollIndicators(.hidden)
    .frame(maxWidth: .infinity)
    .background(.background.secondary)  
  }
  
}
```

```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
  @Query var birdFood: [BirdFood]
 
  var body: some View {
    ScrollView {
      VStack(spacing: 10) {
        if let (birdFood, product) = birdFood.bestValue {
          ProductView(id: product.id) {
            BirdFoodProductIcon(
              birdFood: birdFood,
              quantity: product.quantity
            )
          }
          .padding()
          .background(.background.secondary, in: .rect(cornerRadius: 20))
        }
      }
      .scrollClipDisabled()
      Text("Other Bird Food")
        .font(.title3.weight(.medium))
        .frame(maxWidth: .infinity, alignment: .leading)
      ForEach(birdFood.premiumBirdFood) { birdFood in
        BirdFoodShopShelf(title: birdFood.name) {
          ForEach(birdFood.orderedProducts) { product in
            ProductView(id: product.id) {
              BirdFoodProductIcon(
                birdFood: birdFood,
                quantity: product.quantity
              )
            }
          }
        }
      }
    }
    .contentMargins(.horizontal, 20, for: .scrollContent)
    .scrollIndicators(.hidden)
    .frame(maxWidth: .infinity)
    .background(.background.secondary)  
  }
  
}
```
use large style to make our best product standout
```swift
import SwiftUI
import StoreKit

struct BirdFoodShop: View {
  @Query var birdFood: [BirdFood]
 
  var body: some View {
    ScrollView {
      VStack(spacing: 10) {
        if let (birdFood, product) = birdFood.bestValue {
          ProductView(id: product.id) {
            BirdFoodProductIcon(
              birdFood: birdFood,
              quantity: product.quantity
            )
          }
          .padding()
          .background(.background.secondary, in: .rect(cornerRadius: 20))
          .padding()
          .productViewStyle(.large)
        }
      }
      .scrollClipDisabled()
      Text("Other Bird Food")
        .font(.title3.weight(.medium))
        .frame(maxWidth: .infinity, alignment: .leading)
      ForEach(birdFood.premiumBirdFood) { birdFood in
        BirdFoodShopShelf(title: birdFood.name) {
          ForEach(birdFood.orderedProducts) { product in
            ProductView(id: product.id) {
              BirdFoodProductIcon(
                birdFood: birdFood,
                quantity: product.quantity
              )
            }
          }
        }
      }
    }
    .contentMargins(.horizontal, 20, for: .scrollContent)
    .scrollIndicators(.hidden)
    .frame(maxWidth: .infinity)
    .background(.background.secondary)  
  }
  
}
```
## Standard styles
* compact
* regular
* large

Can also change style of whole storeview.  Even create custom styles.
```swift
StoreView(ids: birdFood.productIDs) { product in 
    BirdFoodShopIcon(productID: product.id)
}
.productViewStyle(.compact)
```

Now we want to do subscriptions
# Subscriptions
SubscriptionStoreView. Can use the others, but this is ideal.



```swift
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
 
    var body: some View {
        Text("Hello, world!") 
    }
  
}
```

fastest way is by providing the groupID from our config file or ASC.

Just declare environment property to access
```swift
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
    @Environment(\.shopIDs.pass) var passGroupID
 
    var body: some View {
        SubscriptionStoreView(groupID: passGroupID)
    }
  
}
```



```swift
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
    @Environment(\.shopIDs.pass) var passGroupID
 
    var body: some View {
        SubscriptionStoreView(groupID: passGroupID) {
            PassMarketingContent() 
        }
    }
  
}
```

can add a container background to make things more visually interesting.


```swift
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
    @Environment(\.shopIDs.pass) var passGroupID
 
    var body: some View {
        SubscriptionStoreView(groupID: passGroupID) {
            PassMarketingContent()
                .lightMarketingContentStyle()
                .containerBackground(for: .subscriptionStoreFullHeight) {
                    SkyBackground()
                }
        }
    }
  
}
```

By default, we add a material layer between controls and background.  Can use style modifier to make it clear instaead.

```swift
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
    @Environment(\.shopIDs.pass) var passGroupID
 
    var body: some View {
        SubscriptionStoreView(groupID: passGroupID) {
            PassMarketingContent()
                .lightMarketingContentStyle()
                .containerBackground(for: .subscriptionStoreFullHeight) {
                    SkyBackground()
                }
        }
        .backgroundStyle(.clear)
    }
  
}
```

multiline action button.

```swift
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
    @Environment(\.shopIDs.pass) var passGroupID
 
    var body: some View {
        SubscriptionStoreView(groupID: passGroupID) {
            PassMarketingContent()
                .lightMarketingContentStyle()
                .containerBackground(for: .subscriptionStoreFullHeight) {
                    SkyBackground()
                }
        }
        .backgroundStyle(.clear)
        .subscriptionStoreButtonLabel(.multiline)
    }
  
}
```

Use material effect for subscription backgrounds.

```swift
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
    @Environment(\.shopIDs.pass) var passGroupID
 
    var body: some View {
        SubscriptionStoreView(groupID: passGroupID) {
            PassMarketingContent()
                .lightMarketingContentStyle()
                .containerBackground(for: .subscriptionStoreFullHeight) {
                    SkyBackground()
                }
        }
        .backgroundStyle(.clear)
        .subscriptionStoreButtonLabel(.multiline)
        .subscriptionStorePicketItemBackground(.thinMaterial)
    }
  
}
```

Use storebutton modifier to declare redeem code button as visible.

```swift
import SwiftUI
import StoreKit

struct BackyardBirdsPassShop: View {
    @Environment(\.shopIDs.pass) var passGroupID
 
    var body: some View {
        SubscriptionStoreView(groupID: passGroupID) {
            PassMarketingContent()
                .lightMarketingContentStyle()
                .containerBackground(for: .subscriptionStoreFullHeight) {
                    SkyBackground()
                }
        }
        .backgroundStyle(.clear)
        .subscriptionStoreButtonLabel(.multiline)
        .subscriptionStorePicketItemBackground(.thinMaterial)
        .storeButton(.visible, for: .redeemCode)
    }
  
}
```

Now, our subscription view matches the feel of the rest of our app.

IAP.  Couple of important pieces we're missing
* unlocking content
* hiding merchandising UI
	* in many cases you don't want to present merchanidising UI to existing customers

# Busines slogic
* handle transaction updates
* validate transaction info with server
* track consumable quantity
* mapping to a model for your view code
[[Meet StoreKit 2]]
[[What's new in App Store Server APIs]]

Just use `.onInAppPurchaseCompletion` on any view?

```swift
BirdFoodShop()
    .onInAppPurchaseCompletion { (product: Product, result: Result<Product.PurchaseResult, Error>) in
        if case .success(.success(let transaction)) = result {
            await BirdBrain.shared.process(transaction: transaction)
            dismiss()
        }
    }
```
In addition to that, there are a few other modifiers.

start: rusn when someone triggers purchase buttons.  Passes the product to be purhased.  ex, dim controls, etc.

```swift
BirdFoodShop()
    .onInAppPurchaseStart { (product: Product) in
        self.isPurchasing = true
    }
```

react to events: Handles events for any SToreKit view descendants
you can add multiple actions to perform
successful transactions come through `Transaction.updates` by default.
Pass `nil` to block ancestor views from handling events and revert to default

Now, whenever backyard thing appears.  Backyard task will load subscription status and call the thing we modify once the task completes.  Just pass the statuses to our business logic.


```swift
subscriptionStatusTask(for: passGroupID) { taskState in
    if let statuses = taskState.value {
        passStatus = await BirdBrain.shared.status(for: statuses)
    }            
}
```

we can be sure our app's UI is always up to date.  If your app offers nonconsumable or non-renewing subscriptions, use 

```swift
currentEntitlementTask(for: "com.example.id") { state in
    self.isPurchased = BirdBrain.shared.isPurchased(
        for: state.transaction
    )
}
```

declare a view as dependent on entilement.
Calls the function whenever the entitlement changes.  Granularly handle loading, failure, or success.

# Adding decorative icons
```swift
ProductView(id: ids.nutritionPelletBox) {
    BoxOfNutritionPelletsIcon()
} placeholderIcon: {
    Circle()
}
```

they show a placeholder icon while loading.  Sometimes the automatic icon doesn't fit, ex square vs round.

If you set an appstore promotion image, you can have the productview use that same image instead of a swiftui view.  Just set the icon to true.

Still provide a swiftui view as a fallback, but this is ignored if it has a promotional icon

[[What's new in StoreKit 2 and StoreKit Testing in Xcode]]
[[23/What's new in AppStore Connect|What's new in AppStore Connect]]

```swift
ProductView(
    id: ids.nutritionPelletBox,
    prefersPromotionalIcon: true
) {
    BoxOfNutritionPelletsIcon()
}
```
can give you the border anyway:
```swift
ProductView(id: ids.nutritionPelletBox) {
    BoxOfNutritionPelletsIcon()
        .productIconBorder()
}
```

Keep in mind there's corresponding api for storeview icons.

# Styling the Product View

I mentioned you can make custom styles.  How?
* appearance
* layout
* interactions

composing standard styles.  Create a type conforming to `ProductViewStyle`.
```swift
struct SpinnerWhenLoadingStyle: ProductViewStyle {

    func makeBody(configuration: Configuration) -> some View {
        switch configuration.state {
        case .loading:
            ProgressView()
                .progressViewStyle(.circular)
        default:
            ProductView(configuration)
        }
    }

}
```

ex we do a custom loading state here.


use
```swift
ProductView(id: ids.nutritionPelletBox) {
    BoxOfNutritionPelletsIcon()
}
.productViewStyle(SpinnerWhenLoadingStyle())
```

you don't need to compose your custom style with a standard style.  other view:

```swift
struct BackyardBirdsStyle: ProductViewStyle {

  func makeBody(configuration: Configuration) -> some View {
    switch configuration.state {
      case .loading: // Handle loading state here
      case .failure(let error): // Handle failure state here
      case .unavailable: // Handle unavailabiltity here
      case .success(let product):
        HStack(spacing: 12) {
          configuration.icon
          VStack(alignment: .leading, spacing: 10) {
            Text(product.displayName)
            Button(product.displayPrice) {
              configuration.purchase()
            }
            .bold()
          }
        }
        .backyardBirdsProductBackground()
    }
  }

}
```

Keep in mind, use the configuration value, not the product value.  Remember when your custom style is built from scratch, appearance and behavior will match your own stuff.

how to make our own loading state for the whole store?  Lift state up into the parent.  Once parent is managing the state of products, it can change appearance of the whole store.  You can pass a product value you've already loaded, cells don't need to load products themselves.

Bring your own caching logic?  Well no, we have some fancy scheme for that.

```swift
@State var productsState: Product.CollectionTaskState = .loading

var body: some View {
    ZStack {
        switch productsState {
        case .loading:
            BirdFoodShopLoadingView()
        case .failed(let error):
            ContentUnavailableView(/* ... */)
        case .success(let products, let unavailableIDs):
            if products.isEmpty {
                ContentUnavailableView(/* ... */)
            }
            else {
                BirdFoodShop(products: products)
            }
        }
    }
    .storeProductsTask(for: productIDs) { state in
        self.productsState = state
    }
}
```

`.storeProductsTask` will pass a collection of product IDs.  Get a state value to handle the states of the async task.  



# Adding auxiliary buttons

* cancellation
* redeem code
```swift
SubscriptionStoreView(groupID: passGroupID) {
   // ...
}
.storeButton(.visible, for: .redeemCode)
```

automatic / visible / hidden.
button kind
* cancellation: show automatically.  May have something to do with presenting as a sheet?  Usually you want to show this, unless you bring your own.
* restorePurchases: hidden by default.
only for subscription
* redeemCode
* signIn: sign in from outside the app store, visible automatically if action declared
* policies: links to privacy policy, etc.

sign in:
```swift
@State var presentingSignInSheet = false

var body: some View {
    SubscriptionStoreView(groupID: passGroupID) {
        PassMarketingContent()
            .containerBackground(for: .subscriptionStoreFullHeight) {
                SkyBackground()
            }
    }
    .subscriptionStoreSignInAction {
        presentingSignInSheet = true
    }
    .sheet(isPresented: $presentingSignInSheet) {
        SignInToBirdAccountView()
    }
}
```

policies
```swift
SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .containerBackground(for: .subscriptionStoreFullHeight) {
            SkyBackground()
        }
}
.subscriptionStorePolicyForegroundStyle(.white)
.storeButton(.visible, for: .policies)
```


# Styling the subscription store view

control styles
* automatic => picks the control
	* iPhone: picker control if multiple plan options
* picker
* prominent picker.  Subscriptions more prominently, ex shadow, ring, etc.
* buttons


```swift
SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .containerBackground(for: .subscriptionStoreFullHeight) {
            SkyBackground()
        }
}
.subscriptionStoreControlStyle(.buttons)
```
customize button labels.  By default, you get a button with caption above.  Can use multiline to put them both in the button

```swift
SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .containerBackground(for: .subscriptionStoreFullHeight) {
            SkyBackground()
        }
}
.subscriptionStoreButtonLabel(.multiline)
```


```swift
SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .containerBackground(for: .subscriptionStoreFullHeight) {
            SkyBackground()
        }
}
.subscriptionStoreButtonLabel(.displayName)
```
compose multiple things together

```swift
SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .containerBackground(for: .subscriptionStoreFullHeight) {
            SkyBackground()
        }
}
.subscriptionStoreButtonLabel(.multiline.displayName)
```

use a viewbuilder to customize each subscription plan
```swift
SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .containerBackground(for: .subscriptionStoreFullHeight) {
            SkyBackground()
        }
}
.subscriptionStoreControlIcon { subscription, info in
    Group {
        let status = PassStatus(
            levelOfService: info.groupLevel
        )
        switch status {
        case .premium:
            Image(systemName: "bird")
        case .family:
            Image(systemName: "person.3.sequence")
        default:
            Image(systemName: "wallet.pass")
        }
    }
    .foregroundStyle(.tint)
    .symbolVariant(.fill)
}
```

add container background.
[[23/What's new in SwiftUI|What's new in SwiftUI]]
* subscription store
* header
* full height




```swift
SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .containerBackground(for: .subscriptionStoreFullHeight) {
            SkyBackground()
        }
}
.subscriptionStoreControlIcon { subscription, info in
    Group {
        let status = PassStatus(
            levelOfService: info.groupLevel
        )
        switch status {
        case .premium:
            Image(systemName: "bird")
        case .family:
            Image(systemName: "person.3.sequence")
        default:
            Image(systemName: "wallet.pass")
        }
    }
   .symbolVariant(.fill)
}
.foregroundStyle(.white)
.subscriptionStoreControlStyle(.buttons)
```

```swift
SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .containerBackground(
            .accent.gradient,
            for: .subscriptionStore
        )
}
```
maybe we want to show to existing subscribers and get them to upgrade to the premium plan.

use `visibleRelationships`.  Only effect when someone is already subscribed.  We can provide a different view for the marketing content.  Use the subscription status task to keep track of level of service, and know which one to present.


```swift
SubscriptionStoreView(
    groupID: passGroupID,
    visibleRelationships: .upgrade
) {
    PremiumMarketingContent()
        .containerBackground(for: .subscriptionStoreFullHeight) {
            SkyBackground()
        }
}
```

* Get started quickly with store view
* Leverage product view for custom layouts
* Build subscription offers with subscription store view
* Take other things to the next level with view modifiers
[[What's new in StoreKit 2 and StoreKit Testing in Xcode]]
[[23/What's new in SwiftUI|What's new in SwiftUI]]

* https://developer.apple.com/documentation/SwiftUI/Backyard-birds-sample
* https://developer.apple.com/documentation/storekit/in-app_purchase/storekit_views
* 