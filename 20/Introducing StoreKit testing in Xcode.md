#storekit #xctest 

Currently, I need to sign into app store connect first.

XC12 - new development and test suite dedicated to storekit and IAP

# New flow
1.  StoreKit Configuration File.  This basically has the IAP info you would have configured in app store connect.  This is an alternative to both `sandbox` and `production`.
2.  Can update configuration file without rebuilding or relaunching.
3.  How to purchase again?  Xcode has a window that shows all transactions.  Can use this to reset state as if the purchase never happened.
4.  Can also use this window to simulate a refund.
5.  Editor->StoreKit->Ask to Buy
6.  Can also simulate "Interrupted purchases".  Idea here is the user needs to update their billing info for example.
7.  Can speed up time in editor menu
8.  Revoke a product [[what's new in in-app purchases]]
9.  Receipt – [[Best Practices and What's New with In-App Purchases - 18]]
	10.  Signed with a different private key
	11.  Certificate exportable from StoreKit configuration editor menu
	12.  Test cert is not part of a certificate change.
	13.  Consider using `#if DEBUG` type stuff


```swift
#if DEBUG
let certificateName = "StoreKitTestCertificate"
#else
let certificateName = "AppleIncRootCertificate"
#endif

//verify receipt using openssl
#if DEBUG
let result = PKCS7_verify(receipt, nil, store, nil, nil PKCS7_NOCHAIN)
#else
let result = PKCS7_verify(receipt, nil, store, nil, nil, nil)
#endif
```


iOS 14, tvOS 14, etc.  Simulator and real devices.
# StoreKitTest Framework
* Full control of the StoreKit in Xcode environment
* Create unit & UI tests with XCTest
* No user interaction is required
* Force subscription renewals

Cases
* successful/failed purchases
* interrupted
* external transactions
* subscription offers

```SKTestSession``` APIs


# Sandbox stuff
* Set up in-app purchases in app store connect
* sandbox appleid
* app receipts are signed by the app store
* server-side receipt validation
* app-store server notifications

app store settings -> sandbox account

Now can view/test subscriptions in sandbox as well.  Can now reset offer eligibility.

[[what's new in in-app purchases]]

Can enable interrupted purchasers in a per-tester way

View and manage subscriptions
* test upgrades, downgrades, cancellation
* reset introductory offer eligibliity
* interrupted purchases
* developer role can create/manage sandbox tester accounts


