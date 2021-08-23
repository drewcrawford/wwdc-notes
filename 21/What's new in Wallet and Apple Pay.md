#applepay
#wallet

# Wallet updates
Bringing identity cards to apple wallet.

New this year, we added multi-pass downloads.

1.  Zip pkpass files together
2.  set extension to `.pkpasses`
3.  use `application/vnd.apple.pkpasses` mime tag

Wallet will automatically hide expired passes, but you can still have them as keepsakes.

# Apple Pay updates
55 countries and regions
Express transit

"Continue" with apple pay button.  Use this when offering applepay in a cart along with other buttons.

New JS implementation for buttons.

Redesigned payment experience.  Written in SwiftUI.  
Conversion enhancements.  Add payment methods without leaving flow
Error handling redesigned.
Payment summary view, now includes icon.  Consider setting a webclip icon.

2x and 3x 120px x120x, 180px x 160px (both 60x60pt).  See HIG.

New icon size requirements for PKPasses
38pt square, 2x & 3x

New total line.  Add a date if the payment occurs later.  

# API enhancements
* Support for shipping date ranges
* Calendar and time zone support

`PKShippingMethod.dateComponentsRange`

Available for shipping, delivery, store pickup, or service pickup

Read-only address.  Inform the user of a pickup location.  Provide address as PKContact, and add shippingContact.  `editingMode=.storePickup` and `requiredShippingContactFields`.

## Coupon codes
* Enter codes after starting transaction
* Guest checkout
* Add discounts as summary items

Perform updates using `didChangeCouponCode` delegate method

* Pre-populate coupon codes
* Display error messages

## Demo

1.  `supportsCouponCode=true`
2.  `request.couponCode="foo"`
3.  `didChangeCouponCode` delegate method
4.  Create PKPaymentSummaryItem to display discount(s).

# Wrap up
* New wallet features
* Updated JavaScript Apple Pay button
* Enhanced payment experience
* New shipping APIs
* Support for coupon codes

* https://developer.apple.com/documentation/passkit/apple_pay/offering_apple_pay_in_your_app
* https://developer.apple.com/documentation/walletpasses/distributing_and_updating_a_pass
* https://developer.apple.com/documentation/apple_pay_on_the_web/displaying_apple_pay_buttons_using_javascript
* https://developer.apple.com/documentation/apple_pay_on_the_web
* https://developer.apple.com/documentation/passkit
* https://developer.apple.com/documentation/passkit/wallet




