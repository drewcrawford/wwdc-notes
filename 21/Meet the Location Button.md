Region-based user notification on apple watch.

Don't need 'always' authorization

[[What's new in watchOS]]

New way to request location that gives people more contorl over their data but gives more functionality to your app.

# State of location authorization.

* allow once
* allow while using
* Don't allow

Last year: precise: on.

Allow once: lose authorization entering the background.

User might select "Don't allow" to prevent prompts.  Then they have to go to settings.

How can we give more control without hindering the UX?

# CLLocationButton

Location arrow at top left will turn blue.

Note: location button is giving your app 'allow once' authorization without the prompt.

How to add this to your app?

# Implementation

Part of CoreLocation framework.  Derived from UIControl.

Four properties.

1.  CLLocationButtonIcon
2.  CLLocationLabel to set label
3.  Corner radius
4.  Font size

Grant a one-time autohrization on your behalf.

SwiftuI, use `LocationButton`.  

# Customization

1.  icon
2.  tintColor
3.  backgroundColor
4.  cornerRadius

If button's width is equal to height, can create a perfect circle when corner radius is half the width.

Create the perfect look.  However, with great power comes great responsibility.

# Restrictions

> While your target action will be called, you won't get authorization

But there will be log messages.

* inappropriate sizes
* alpha
* contrast ratio between `tintColor` and `backgroundColor`

Shown as a buildtime issue for IB.

Button's text length can change depending on language.  Size of text may outgrow the button.  

Test location button with small/large text sizes or languages.

# Just like 'allow once'

If your app is already authorized, using location button will not change existing state and will behave like any other UIButton.

[[What's new in watchOS]]



