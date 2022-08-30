New features available for testing IAP in StoreKit.
# StoreKit updates in Xcode 14
In WWDC 2020, we introduced SK testing in xcode.  Allowing you to start testing IAP in xcode.  This year we have updates.

You can create a storekit configuration file in xcode, and start testing your IAP without an ASC app.  When ready to configure, we're introducing a brand new feature to let you use the same purchase products from xcode.

Configure your IAP once, and use same ocnfiguration locally in xcode, inside unit tests, in the sandbox environment, and in production.

## App Store Connect Sync.
1.  Configure in ASC
2. Create synced configuration file in xcode

Make change from ASC=>Xcode.

Can also convert a synced configuration into a local operation.  **This is a one way operation.**

## Demo
"StoreKit Configuration" file type from picker.

Synced file is in a read-only state.  To make changes we have to open ASC.  Then press sync butto in bottom lefthand corner of xcode.  

Copy data to a local file and make changes inside xcode.  In addition to copying items, we acn convert an entire synced file to a local, editable file.  Open our synced file, go to the editor menu, and lcick on "Convert to Local StoreKit Configuration".

You can't undo this operation after undoing the file.  To sync again, need a new file.

Configure our testing environment.  Open scheme editor, select run option, Options, StoreKit configuration.
* None => sandbox
* files => xcode environment

Both environments now use the exact same product and subscription metadata.  

In XC14, products from SK configuration files will load right into previews.

```swift
VStack(alignment: .leading) {
    Text(subscription.displayName)
        .font(.headline.weight(.semibold))
    Text(subscription.description)
}
```

Powerful new tools in the SK transaction manager.  We can open the transaction manager from debug bar.  It looks like an arrow bending around toward you.

Jump buttons take you to the configuration file.Filter transactions.By id, by purchase date, etc.

## Wrapup

# Advanced subscription cases
* Refund requests
* offer codes
* price increases
* billing retry and grace period

## Refund requests
Our support view lets us choose a transaction to refund.
```swift
struct RefundView: View {
    @State private var selectedTransactionID: UInt64?
    @State private var refundSheetIsPresented = false
    @Environment(\.dismiss) private var dismiss
    var body: some View {
        Button {
            refundSheetIsPresented = true
        } label: {
            Text("Request a refund")
                .bold()
                .padding(.vertical, 5)
                .frame(maxWidth: .infinity)
        }
        .buttonStyle(.borderedProminent)
        .padding([.horizontal, .bottom])
        .disabled(selectedTransactionID == nil)
        .refundRequestSheet(
            for: selectedTransactionID ?? 0,
            isPresented: $refundSheetIsPresented
        ) { result in
            if case .success(.success) = result {
                dismiss()
            }
        }
    }
}
```

Refund requests will take time to process.  When xcode in sandbox, immediately refunds the transaction.  Inspector shows revocation reason and date.  Test refudns by clicking the refund button in transaction manager.

How to use SK to handle refunded transactions

```swift
for await update in Transaction.updates {
    let transaction = try update.payloadValue
  
    if let revocationDate = transaction.revocationDate,
  	   let revocationReason = transaction.revocationReason {
        print("\(transaction.productID) revoked on \(revocationDate)")
       
        switch revocationReason {
        case .developerIssue: <#Handle developer issue#>
        case .other: <#Handle other issue#>
        default: <#Handle unknown reason#>
        }
        
        <#Revoke access to the product#>
    }
    <#...#>
}
```

Use revocationdate and revocationReason.  

|            | xcode | sandbox |
| ---------- | ----- | ------- |
| iOS/iPadOS | 15.2  | 15.0    |
| macOS      | 12.1  | 12.0        |

## Offer codes
Select a subscription in xcode config and press the `+` under the offer codes table.  Configure the offer.  If syncing with ASC, your ASC offers will show up here automatically.

```swift
struct SubscriptionPurchaseView: View {
    @State private var redeemSheetIsPresented = false
        
    var body: some View {
        Button("Redeem an offer") {
            redeemSheetIsPresented = true
        }
        .buttonStyle(.borderless)
        .frame(maxWidth: .infinity)
        .padding(.vertical)
        .offerCodeRedeemSheet(isPresented: $redeemSheetIsPresented)
    }

}
```

in xcode, redemption is more streamlined - you can just tap each offer?
In renewals tab, see the explanation:
1.  Renewals with introductory offer
2. offer code
3. final price

Great way to offer flexible promotions to future and existing subscribers.  Easier than ever to get started using offer codes.

```swift
for await verificationResult in Transaction.updates {
    guard case .verified(let transaction) = verificationResult else {
        <#Handle failed verification#>
    }
    <#Handle updated transaction#>
}

for await updatedStatus in Product.SubscriptionInfo.Status.updates {
    guard case .verified(let renewalInfo) = updatedStatus.renewalInfo else {
        <#Handle failed verification#>
    }
    <#Handle updated status#>
}
```

See if there is an offer applied to the current transaction.  In this case, will be `introductory`.  On the renewalInfo we can see `offerType` to see what will be present in the next renewal.  This will intiially be `introductory`.  After 2 subscription periods, we will see this switch to `code` as we have a code offer set.


```swift
for await status in Product.SubscriptionInfo.Status.updates {
    let transaction = try status.transaction.payloadValue
    let renewalInfo = try status.renewalInfo.payloadValue
    
    if let currentOfferType = transaction.offerType {
        switch currentType {
        case .introductory: <#Handle introductory offer#>
        case .promotional:  <#Handle promotional offer#>
        case .code:         <#Handle offer for codes#>
        default:            <#Handle unknown offer type#>
        }
        self.hasCurrentOffer = true
    }

    <#...#>

}
```

```swift
for await status in Product.SubscriptionInfo.Status.updates {
    let transaction = try status.transaction.payloadValue
    let renewalInfo = try status.renewalInfo.payloadValue
    
    <#Check active current offer#>
    
    if let nextOfferType = renewalInfo.offerType {
        switch currentType {
        case .introductory: <#Handle introductory offer#>
        case .promotional: <#Handle promotional offer#>
        case .code:
            print("Customer has \(renewalInfo.offerID) queued")
            <#Handle offer for codes#>
        default: <#Handle unknown offer type#>
        }
        self.hasQueuedOffer = true
    }
    <#...#>
}
```

Configur eusing xcode 13.3 or later
test in IOS 15.4 or later.

## Price increases
Increase price for subscription in configuration.  This step is optional.  

Transaction manager => request price increase consent.  Wee in the transaction manager is now in "price increase penidng" state.  Sheet apperas asking for consent.  This appears on its own, but can customize with messages API.
```swift
private var pendingMessages: [Message] = []

private func updatesLoop() {
    for await message in Message.messages {
      if <#Check if sensitive view is presented#>,
         let display: DisplayMessageAction = <#Get display message action#> {
           try? display(message)
      }
      else {
        pendingMessages.append(message)
      }
    }
}
```

Make sure we don't have a sensitive view prevented.  Otherwise we will use the `DisplayMessageAction` to present the message.

Each time we press the button in the transaction manager, we get the message again.  

## Demo
While we could accept the price increase or cancel the subscription in the sheet, user might resopnd via external source like email.

Use "Approve" and "Decline" buttons in transaction manager.  

```swift
for await status in Product.SubscriptionInfo.Status.updates {
    let renewalInfo = try status.renewalInfo.payloadValue

    if renewalInfo.priceIncreaseStatus == .agreed {
        print("Customer consented to price increase")
        <#Handle consented to price increase#>
    }
    if renewalInfo.expirationReason == .didNotConsentToPriceIncrease {
        print("Customer did not consent to price increase")
        <#Handle expired due to not consenting to price increase#>
    }

    <#...#>

}
```

Check `priceIncreaseStatus` value.  If customer cancels, we will be able to detect this `.didNotConsentToPriceIncrease`.

## Unit tests

```swift
let session: SKTestSession = try SKTestSession(configurationFileNamed: "<#Configuration name#>")
session.disableDialogs = true

<#Purchase a subscription#>

var transaction: SKTestTransaction! = session.allTransactions().first
session.requestPriceIncreaseConsentForTransaction(identifier: transaction.identifier)

transaction = session.allTransactions().first
XCTAssertTrue(transaction.isPendingPriceIncreaseConsent)

<#Assert app updates for pending price increase#>

// Write a test case for consenting and cancelling due to price increase:

session.consentToPriceIncreaseForTransaction(identifier: transaction.identifier)

// OR

session.declinePriceIncreaseForTransaction(identifier: transaction.identifier)
session.expireSubscription(productIdentifier: "<#Product ID#>")

<#Assert app updates for finished price increase#>
```

Configure using Xcode 13.3 or later

|                | status | message |
| -------------- | ------ | ------- |
| iOS and iPadOS | 15.4   | yes     |
| macOS          | 12.3   | no      |
| watchOS        | 8.5    | no      |
| tvOS           | 15.4   | no        |

## Billing retry and grace period
Error occurred when tryign to renew a subscription, such as expired credit card.  App store will attempt to fix the issue and recover the subscription.  

To simualte billing issues, we enable "billing retry on renewal".  Editor=>enable billing grace period.  Editor=>subscription renewal rate=>...

Now we can subscribe to the whatever.

1.  Enter billing grace period.
2. Standard  billing retry state.
3. At any time we can use resolve issue button to resolve the issue.
4. Now we get a new transaction.
5. So long as we have these options enabled, each new transaction will continue to enter biling retry.

```swift
for await status in Product.SubscriptionInfo.Status.updates {
    let renewalInfo = try status.renewalInfo.payloadValue

    if let gracePeriodExpirationDate = renewalInfo.gracePeriodExpirationDate,
       gracePeriodExpirationDate < .now {
        print("In grace period until \(gracePeriodExpirationDate)”)
        <#Allow access to subscription#>
    }
    else if renewalInfo.isInBillingRetry {
        <#Handle billing retry#>
    }

    <#...#>

}
```

`.gracePeriodExpirationDate`.  To check for billing retry, we just check `.isInBillingRetry`.

Give them a deep link:

```swift
struct SubscriptionStatusView: View {
    let currentSubscription: Product
    let status: Product.SubscriptionInfo.Status
    @Environment(\.openURL) var openURL
    var body: some View {
        Section("Your Subscription") {
            <#...#>
            if status.state == .inBillingRetryPeriod || status.state == .inGracePeriod {
                VStack {
                    Text("""
                    There was a problem renewing your subscription. Open the App Store to
                    update your payment information.
                    """)
                    Button("Open the App Store") {
                        openURL(URL(string: "https://apps.apple.com/account/billing")!)
                    }
                }
            }
        }
    }
}
```

You will see entitlements in the grace period:
```swift
for await entitlement in Transaction.currentEntitlements {
    <#Grant access to product#>
}
```

## unit testing
```swift
let session: SKTestSession = try SKTestSession(configurationFileNamed: "<#Configuration name#>")
session.billingGracePeriodIsEnabled = true
session.shouldEnterBillingRetryOnRenewal = true

<#Purchase a subscription#>

wait(for: [<#XCTExpectation#>], timeout: 60)

let transaction: SKTestTransaction! = session.allTransactions().first
XCTAssertTrue(transaction.hasPurchaseIssue)

<#Assert app still allows access to subscription due to grace period#>

wait(for: [<#XCTExpectation#>], timeout: 60)

<#Assert app detects billing retry and no longer allows access to subscription#>

session.resolveIssueForTransaction(identifier: transaction.identifier)

<#Assert app allows access to subscription#>
```

Simulate the app store recovering the subscription.

|            | xcode 13.3 | sandbox |
| ---------- | ---------- | ------- |
| iOS/iPadOS | 15.4       | 16.0    |
| macOS      | 12.3       | no      |
| watchOS    | 8.5        | no      |
| tvOS       | 15.4       | no      |
| Server     | N/A        | yes        |

## Wrap up
* refund requests
* offer codes
* price increases
* billing retry and grace period

[[What's new with in-app purchases]]


Other new:
* new subscruiption renewal rates
* in-app manage subscriptions sheet
* ad network attribution

[[What's new in SKAdNetwork]]

# Sandbox updates
How new features with SK testing can help yu text more complex IAP implementations.

Many of you rely on app store sandbox environment to test iap and server implementation.  Excited to share new enhancements.  Test your app/server in an online test environment.

## Sandbox apple ID creation
Set up ID in ASC.

users and access => Sandbox Testers.
Create a new tester.  We've streamlined creation process by removing several fields.  Only minimum info, so you can move forward with creating your account.

Use a `+` symbol in your email address so you don't need to createa  new email for each tester.
We made strong passwords tedious.

Inline suggestisons for making your password more secure.  WE hope streamlined password creation will spend less time on accounts.
## Appstore Connect API
We've been adding features to sandbox.  Many features are accessible in ASC.

Later this year, several of these are coming to API
* query for sandbox apple IDs
* clear IAP history
* set interrupted purchase state

Faster testing via sandbox accounts.  Automation clients for testing tools.
## Building failure simulation
In 2018 we announced billing retry and grace period for autorenewal.  Since laucnhing in 2019, BGP has allowed you to recover 300 million days of paid service to your customers.

While many of you are handling this in prod, we want to provide more testing scenarios.  

* Sandbox Account Settings
* Foregruond buy and subscription renewal failure
* Verify status with App Store Server API

[[Engineering Subscriptions - 18]]

A switch in the new Sandbox Account Settings to simulate failed IAP attempt.  With this enabled, foregruondi n-app purchases will fail.  Matches the behavior when customer's payment method is declined.

You can test in-app messaging for your customers who are having billing problems.  These will be reflected in IAP receipts, with v2 notifications.

1.  SUBSCRIBED
2. DID_RENEW, etc.
3. DID_FAIL_TO_RENEW (billing retry)
4. DID_RENEW, BILLING_RECOVERY (if failure simulation is disabled)
5. EXPIRED, BILLING_RETRY (if not disabled)


DID_FAIL_TO_RENEW, GRACE_PERIOD
GRACE_PERIOD_EXPIRED

You acn verify subscription state by decoding payload.

Once you've completed testing, disable the switch in sandbox account settings.  We hope this helps

# Wrap up
* app store connect syncing
* New testing capabilities in xcode
* Subscription management for billing issues

[[Manage in-app purchases on your server]]
[[What's new with in-app purchases]]



* https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase/subscriptions_and_offers/implementing_offer_codes_in_your_app
* https://help.apple.com/app-store-connect/#/dev6a098e4b1
* https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase/subscriptions_and_offers/reducing_involuntary_subscriber_churn
* https://developer.apple.com/documentation/storekit/in-app_purchase/testing_in-app_purchases_with_sandbox
* https://developer.apple.com/documentation/Xcode/setting-up-storekit-testing-in-xcode
* https://developer.apple.com/documentation/storekit/in-app_purchase/original_api_for_in-app_purchase/subscriptions_and_offers/handling_subscriptions_billing
* https://developer.apple.com/app-store/subscriptions/
