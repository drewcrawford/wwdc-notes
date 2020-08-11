# API enhancements
`PKPaymentPass`.  We continue to expand featureset, so instead `PKSecureElementPass`.

New button types.

Button style can now switch with system dark mode.  `payenmtButtonStyle:.automatic`

# Supporting apple pay in app clips
## Best practices
* use apple pay where possible
* allow guest checkout
* Simplicity
* "light" c.f. johnny ive.  Load assets on-demand.
# enhancing across platforms
Can now modify PKPaymentButton corner radius.

Redacted billing address for digital purchases.

Today, we're bringing apple pay to catalyst and native mac apps.

[[Apple pay on the web - 16]]

Nowadays, we use `apple-pay-gateway.apple.com` the fixed server.

On mac, need to  implement some delegate methods to configure which window to present over.

# Tiered WebKit
WKWebViews may further restrict APIs.
Apps displaying WKWebView may be restricted in the APIs they invoke.
Apple Pay is already only supported on webpages that do not inject script.

# Contacts Formatting Improvements
Historically, we haven't validated contact data, shipping addresses, etc.  Merchants do whatever, and sometimes they reject the address a user inserted.

We are standardizing the address we give you.

Raising formatting errors earlier in the payment flow.  Users find out they need to correct information prior to authentication.

We recommnd using our error APIs so users understand what they did wrong

Initially making these changes to 4 countries

# Adding cards to apple pay
Issuer extensions to add a card within wallet.

App must have a non-UI extension that reports status.
Extension principal object must sublcass `PKIssuerProvisioningExtensionhandler`
Requires entitlement

UI extension to handle reauthentication as well

`apple-pay-inquiries@apple.com`



