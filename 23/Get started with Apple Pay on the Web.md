Adding Apple Pay to your website elevates your customer experience. Learn how to present Apple Pay as a payment option, validate your merchant session, and authenticate and process payments. You'll also find out how to configure your environment, set up transactions using the Apple Pay demo site, and test your implementation.

1.  Customer
2. merchant website
3. merchant server
4. payment service provider
5. acquirere
6. payment network
7. issuer
# Payment request API
* cross-browser w3c standard
* secure, consistent, accessible

# Configure environment
* Payment provider supports apple pay
* valid ssl cert
* tls 1.2+
* https://

sign up on developer.apple.com
Create a merchant identifier -> merchant.com.example
create payment processing certificate
Create merchant identity certificate.
verify domain


# Implement Apple Pay
1.  Customer device
2. create payment request on merchant website
3. request merchant session on merchant server
4. apple pay generates the session, sends to merchant server
5. complete validation on website
6. customer device confirms payment
7. generates payment data
8. sent to apple pay server and returned.  
9. Payment data sent to website by device.
10. You pass to PSP to decrypt, charge, return payment status
11. website confirms payment outcome, dismiss sheet.

can be divided roughly into these buckets:
## presenting AP

display whenever your customer is using supported device.  `window.ApplePaySession` and `canMakePayments`

* JS or CSS
* style
* type
* localization
recurring payments, automatic reload, etc.

ensure country code is set correctly, esp for psd2 markets.

can include contact details, etc.

modifiers.  Only under certain conditions, e.g. surcharge for credit cards, etc.

recurringPaymentRequest.

Create all these together to `new PaymentRequest(methods, details, options);`

request.canMakePayment().  Checks that the customer is on device and browser, plus card available.


## merchant validation

for every web transaction, important , etc.

1.  website
2. server
3. apple pay server

onMerchantValidation -> call merchant server.  apple-pay-gateway.apple.com.

request must come from the merchant server, to keep your identity cert from other parties.

send JSON payload to apple server using two-way TLS.  Use merchant identity certificate.  Returns merchant session object to server.  Do not modify otherwise validation will fail.

Pass back to browser.  Then you pass that to the completion method to complete the validation process.

onPaymentMethodChange must be called within 30s.  

At first, we send only info necessary for taxes/shipping.  Then you can calculate the total.  

## payment authentication
## payment processing
complete the payment.  `response.complete("success")`.  

can also use `fail`.  

# Test solution
developer.apple.com/apple-pay/ssandbox-testing/

we have ~~fake~~ sandbox credit cards for you to use.

we recommend testing real cards.

# Next steps
* explore apple pay demo site
* review merchant integration guide
* check out apple pay section of HIG

# Resources
* https://developer.apple.com/documentation/passkit/apple_pay
* http://developer.apple.com/apple-pay/Apple-Pay-Merchant-Integration-Guide.pdf
* https://applepaydemo.apple.com/
* https://developer.apple.com/apple-pay/sandbox-testing/
* https://developer.apple.com/documentation/Technotes/tn3103-apple-pay-on-the-web-troubleshooting-guide
