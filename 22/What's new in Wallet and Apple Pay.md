#wallet  #applepay 

72 countries, 1m transactions per day.


# Updates
Tap to Pay on iPhone => iOS 15.4.  Secure, private, and easy way to accept contactless payments.  Integrate seamlessly/securely accept payments.  Apple pay, ocntactless credit/debit, and other digital wallets.

In MacOS 13, we redesigned apple pay experience.  New sheet design.  This year we're bringing it to macOS.  SwiftUI.

New SwiftUI APIs.
* add to apple wallet buttons, apple pay buttons

```swift
@State var addedToWallet: Bool

@ViewBuilder private var airlineButton: some View {
    if let pass = createAirlinePass() {
        AddPassToWalletButton([pass]) { added in
            addedToWallet = added
        }
        .frame(width: 250, height: 50)
        .addPassToWalletButtonStyle(.blackOutline)
    } else {
        // Fallback
    }
}
```

Can make it wider, taller, etc.  

Let's see how to add apple pay button.  First, cretae a payment request with `PKPaymentRequest`.

```swift
// Create a payment request
let paymentRequest = PKPaymentRequest()
// ...

// Create a payment authorization change method
func authorizationChange(phase: PayWithApplePayButtonPaymentAuthorizationPhase) { ... }

PayWithApplePayButton(
    .plain,
    request: paymentRequest,
    onPaymentAuthorizationChange: authorizationChange
) {
    // Fallback
}
.frame(width: 250, height: 50)
.payWithApplePayButtonStyle(.automatic)
```

In total, there are 17 different labels, so you can customize the payment button to align with your usecase.  iOS, iPadOS, macOS, watchOS.

# Multi-merchant payments
* Multiple payment tokens per transaction
* Marketplaces
* travel bookinsg
* ticketing services

You might imagine the travel agency will charge her $500, and pay the other companies involved.  Typically though, the travel agency passes along the charges to each company.

Not great for allison's privacy and security.  Now, with new multimerchant payment API, request a payment token for each merchant involved in the transaction.  The multipel companies can charge for the relevant amounts she authorizes.

Payment sheet shows breakdown of submerchants.  
```swift
// Create a payment request
let paymentRequest = PKPaymentRequest()
// ...

// Set total amount
paymentRequest.paymentSummaryItems = [
    PKPaymentSummaryItem(label: "Total", amount: 500)
]

// Create a multi token context for each additional merchant in the payment
let multiTokenContexts = [
    PKPaymentTokenContext(
        merchantIdentifier: "com.example.air-travel",
        externalIdentifier: "com.example.air-travel",
        merchantName: "Air Travel",
        merchantDomain: "air-travel.example.com",
        amount: 150
    ),
    PKPaymentTokenContext(
        merchantIdentifier: "com.example.hotel",
        externalIdentifier: "com.example.hotel",
        merchantName: "Hotel",
        merchantDomain: "hotel.example.com",
        amount: 300
    ),
    PKPaymentTokenContext(
        merchantIdentifier: "com.example.car-rental",
        externalIdentifier: "com.example.car-rental",
        merchantName: "Car Rental",
        merchantDomain: "car-rental.example.com",
        amount: 50
    )
]
paymentRequest.multiTokenContexts = multiTokenContexts
```

Use the same external id for the merchant any time you request a payment for that merchant.

For web, see documentation.

# Automatic payments
View and manage automatic payments in Wallet.
* recurring payments
* Automatic reload payments
* Introducing apple pay merchant tokens

Imagine julie is paying for a book club membership on iPhone.  Book club receives a payment token based on the device.  What happens if julie gets a new iPhone?

With automatic payments, book club receives an apple pay merchant token.  jTied to julie's apple ID, rather than her iPhone, providing better assurances for ongoing transactions.

## Recurring payments
* fixed or variable amount charged on a regular schedule.
* End on a specific date or until canceled
* Trial or introductory period
* Subscriptions, isntallments, regular billing

```swift
// Specify the amount and billing periods
let regularBilling = PKRecurringPaymentSummaryItem(label: "Membership", amount: 20)

let trialBilling = PKRecurringPaymentSummaryItem(label: "Trial Membership", amount: 10)

let trialEndDate = Calendar.current.date(byAdding: .month, value: 1, to: Date.now)
trialBilling.endDate = trialEndDate
regularBilling.startDate = trialEndDate

// Create a recurring payment request
let recurringPaymentRequest = PKRecurringPaymentRequest(
    paymentDescription: "Book Club Membership",
    regularBilling: regularBilling,
    managementURL: URL(string: "https://www.example.com/managementURL")!
)
recurringPaymentRequest.trialBilling = trialBilling

recurringPaymentRequest.billingAgreement = """
50% off for the first month. You will be charged $20 every month after that until you cancel. \ You may cancel at any time to avoid future charges. To cancel, go to your Account and click \ Cancel Membership.
"""

recurringPaymentRequest.tokenNotificationURL = URL(
    string: "https://www.example.com/tokenNotificationURL"
)!

// Update the payment request
let paymentRequest = PKPaymentRequest()
// ...
paymentRequest.recurringPaymentRequest = recurringPaymentRequest

// Include in the summary items
let total = PKRecurringPaymentSummaryItem(label: "Book Club", amount: 10)
total.endDate = trialEndDate
paymentRequest.paymentSummaryItems = [trialBilling, regularBilling, total]
```

Your total for the payment request is the first amount hte customer will be charged.

## Automatic reload payments
* fixed top-up amount payment
* Charged whenever account balance drops below a set threshold amount
* store cards, pre-paid accounts

```swift
// Specify the reload amount and threshold
let automaticReloadBilling = PKAutomaticReloadPaymentSummaryItem(
    label: "Coffee Shop Reload",
    amount: 25
)
reloadItem.thresholdAmount = 5

// Create an automatic reload payment request
let automaticReloadPaymentRequest = PKAutomaticReloadPaymentRequest(
    paymentDescription: "Coffee Shop",
    automaticReloadBilling: automaticReloadBilling,
    managementURL: URL(string: "https://www.example.com/managementURL")!
)

automaticReloadPaymentRequest.billingAgreement = """
Coffee Shop will add $25.00 to your card immediately, and will automatically reload your \
card with $25.00 whenever the balance falls below $5.00. You may cancel at any time to avoid \ future charges. To cancel, go to your Account and click Cancel Reload.
"""

automaticReloadPaymentRequest.tokenNotificationURL = URL(
    string: "https://www.example.com/tokenNotificationURL"
)!

// Update the payment request
let paymentRequest = PKPaymentRequest()
// ...
paymentRequest.automaticReloadPaymentRequest = automaticReloadPaymentRequest

// Include in the summary items
let total = PKAutomaticReloadPaymentSummaryItem(
    label: "Coffee Shop",
    amount: 25
)
total.thresholdAmount = 5
paymentRequest.paymentSummaryItems = [total]
```

For web, see documentation.

## Best practices
* Include automatic payments in your summary items
* Total amount should be the first amount charged
* Keep billing agreement text short
* Billing agreemnet text should not replace normal billing and legal agreements
	* If you have a legal agreement to show, maybe display it before the payment sheet?
* One automatic payment per transaction
* Not compatible with multi-merchant payments
* Specify a token notification URL to receive life cycle notifications

# Order tracking
New in iOS 16, order trackign allows users to track orders placed with participating merchants.

Shipping, pickup instructions, etc.

* Order Type ID
	* identifies your organization as an entity that provides order information
	* multiple order type IDs to provide ordering information on behalf of multiple merchants
* Certificate
* Packages
	* Metadata, order information, etc.
	* shipping, pickup, etc.
	* images => logo, line item
	* Localizations
	* must be cryptographically signed
	* compressed into a `.order` file
	* see sample code

## Adding an order
1.  App or webpage receives payment information
2. sends to server
3. creates order
4. returns order details
5. Enables the device to asynchronously request the order from server
6. Returns order package to the device

Assign an order ID
* scoped to your order type ID
* Generate an authentication token

```swift
func onAuthorizationChange(phase: PayWithApplePayButtonPaymentAuthorizationPhase) {
    switch phase {
    // ...
    case .didAuthorize(let payment, let resultHandler):
        server.createOrder(with: payment) { serverResult in
            guard case .success(let orderDetails) = serverResult else { /* handle error */ }
            let result = PKPaymentAuthorizationResult(status: .success, errors: nil)
            result.orderDetails = PKPaymentOrderDetails(
                orderTypeIdentifier: orderDetails.orderTypeIdentifier,
                orderIdentifier: orderDetails.orderIdentifier,
                webServiceURL: orderDetails.webServiceURL,
                authenticationToken: orderDetails.authenticationToken,
            )
            resultHandler(result)
        }
    }
}
```

Your app sends the payment information to your server and asks it to create an order.

```js
paymentRequest.show().then((response) => {
    server.createOrder(response).then((orderDetails) => {
        let details = { };
        if (response.methodName === "https://apple.com/apple-pay") {
            details.data = {
                orderDetails: {
                    orderTypeIdentifier: orderDetails.orderTypeIdentifier,
                    orderIdentifier: orderDetails.orderIdentifier,
                    webServiceURL: orderDetails.webServiceURL,
                    authenticationToken: orderDetails.authenticationToken,
                },
            };
        }
        response.complete("success", details);
    });
});
```

## Updating an order
1.  When device registers for updates to order
2. Server adds registration
3. Server updates the order later, uses the registration information to notify devices
4. Sends push notification
5. Device requests order to see updated info

Order information is sensitive.  Directly between devices and your server.  E2E encrypted.

## Best practices
* associate your app to manage notifications (disable order tracking if you have an app?)
* Localize according to your customer preferences
* Be mindful of the order package size
* Update orders promptly
* follow the HIG

Great way to enhance the post-purchase experience.  With automatic updates, your customers will always be up tod ate about the status of their orders.

# Identity verification

Launched in 15.4.  

Apps and app clips can request information.  Verify age or identity.  App requests information, user approves requests, sends response to server for decryption/verification.

1. Name
2. address
3. DOB
4. Photo (aka portrait)
5. Issuing authority
6. Document number, expiration date
7. driving privileges if any

you can ask
* age at least 18?
* 21?
* 65?
* \_?

Sheet will show what information you're requesting, how long you intend to store, etc.  User makes an informed deceision about whether to share the information with your app.

Response signed by id's issuing authority.  Verify information int he responsse is authentic.  Issugin authority creates the id but is not involved.

* entitlement is required
* merchant ID, encryption certificate

## Verification flow
4 steps
1.  REquests via PassKit API
2. User prompted to approve request
3. App receives encrypted response
4. App server decrypts and verifies response

## PassKit API
For SwiftUI

```swift
@ViewBuilder var verifiyIdentityButton: some View {
    VerifyIdentityWithWalletButton(
        .verifyIdentity,
        request: createRequest(),
    ) { result in
        // ...
    } fallback: {
        // verify identity another way
    }
}
```

Like other buttons, we have a lot of configuration options
specify PKIdentityREquest

```swift
func createRequest() -> PKIdentityRequest {
    let descriptor = PKIdentityDriversLicenseDescriptor()
    descriptor.addElements([.age(atLeast: 18)],
                            intentToStore: .willNotStore)
    descriptor.addElements([.givenName, .familyName, .portrait],
                            intentToStore: .mayStore(days: 30))

    let request = PKIdentityRequest()
    request.descriptor = descriptor
    request.merchantIdentifier = // configured in Developer account
    request.nonce = // bound to user session
}
```


```swift
@ViewBuilder var verifiyIdentityButton: some View {
    VerifyIdentityWithWalletButton(
        .verifyIdentity,
        request: createRequest(),
    ) { result in
        // ...
    } fallback: {
        // verify identity another way
    }
}
```
Result object with the outcome of the request.

```swift
@ViewBuilder var verifiyIdentityButton: some View {
    VerifyIdentityWithWalletButton(
        .verifyIdentity,
        request: createRequest(),
    ) { result in
        switch result {
        case .success(let document):
            // send document to server for decryption and verification
        case .failure(let error):
            switch error {
            case PKIdentityError.cancelled:
                // handle cancellation
            default:
                // handle other errors
            }
        }
    } fallback: {
        // verify identity another way
    }
}
```

Can also use PKIdentity button and PKIdentityAuthorizationController to accomplish the same thing.

What does your server do?
* RFC 8949
* RFC 9180
* ISO/IEC 18013-5

CBOR encoded encryption envelope (RFC 8949).  Binary data to encode objects.  Envelope contains metadata needed.

Decrypt with HPKE - RFC 9180.  Server will decrypt with its private key.
Decrypted text is mdoc response (ISO 18013-5).  This contains the elements you requested, a number of security features you need to validate.  Your server will perform the decryption and validation itself.  Neither apple's servers nor the issuing authority is involved.

## Session transcript
CBOR structure that binds a resopnse payload to a specific request for specific app.  Server builds and uses during decryption and validation.

Contains nonce and merchant ID in your request.  Team ID of your developer team, and sha256 hash of certificate public key.

Server should check that the inputs you're using are all valid.  nonce hasn't been used, isn't tied to the current user, matches your developer account, etc.

Now you need transcript, envelope, private key for the certificate.  Ensure your private key stays private.  **Don't include it in your app**.

After decrypting the data, mdoc response.  This has 2 signatures.  Issuer signature, device signature.  **Check both signatures**.

issuer certificate => from issuing authority.  By checking this signature you're verifying that the data came from real issuinga uthority.  Not only is it valid, but it's signed by an issuer you trust.  See documentation.

Device signature => key in the secure element of the user's element.  Provides response came from same iPhone as the issuing authority originally issued the id to.  

Finally, can use the data elements.  never use these elements without *first* verifying *both* signatures.

## Testing
* iOS simulator
	* API returns mock response
	* similar to a real one but lacks real signatures
* On an iPhone with a test profile
	* See docs for more details
	* Server should never treat mock responses like a real one
* Example response


# Wrap up
* multi-merchant payments
* automatic payments
* order tracking
* identity verification


* https://developer.apple.com/forums/tags/wwdc2022-10041
* https://developer.apple.com/forums/create/question?&tag1=28&tag2=257&tag3=345030
* https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentrequest
* https://developer.apple.com/documentation/merchanttokennotificationservices
* https://developer.apple.com/documentation/walletorders
* https://developer.apple.com/design/human-interface-guidelines/technologies/wallet/designing-order-tracking
* https://developer.apple.com/documentation/passkit/wallet/verifying_wallet_identity_requests
* https://developer.apple.com/documentation/passkit/wallet/requesting_identity_data_from_a_wallet_pass
* https://applepaydemo.apple.com
* https://developer.apple.com/documentation/apple_pay_on_the_web
* https://developer.apple.com/documentation/passkit
* https://developer.apple.com/documentation/passkit/apple_pay
