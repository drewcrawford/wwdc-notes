#appclips 

Fast and focused experience on a specific task.  User can upgrade to your app after they complete a task.

# App Clip card in Safari
Display smart app banner in safari or view controller.  In iOS 15, can configure the meta tag to show full-size app clip card in webpage.  User can open your applcip from there.

Safari will remember the choice if the user chooses not to see the card.

Add `app-clip-display=card` to the meta tag.

For apps using safarivewcontroller, will show card as part of app's UI.  
# Test with local experiences
A user's appclip experience starts with appclip card.  Card shows you title, subtitle, what action it can take, header image, etc.

You can create an appclip card in appstore connect.  Specify locations associated with your appclip.  

[[What's new in appstore connect]]
[[Configure and link your app clips]]

You might not have registered any experiences during development.  Sometimes you want to test end-to-end flow.

Experience you entered in on your testing device.  Launch with regular invocation methods.  Developer settings => local experiences.  Specify URL, bundleID, etc.  

NFC tag to launch expeirence.  Local experiences support QR, NFC, appl clip code, safari, messages.  


# App clip code
New (iOS 14.3) visual code to discover app clip experiences in the real world.  1 billion devices ready for your code.

Code always links to an app clip.  Each code encodes a unique URL.  iOS encodes the URL before sending the appclip.  Your app doesn't need to do anything special.

Two types of app clip codes.  NFC integrated or scan only.  NFC integrated => tap or scan with camera.  

Use scan only if the code may be placed out of reach of the customer, or when NFC is not appropriate.  For example, when you display digitally in a marketing email.  You can customize a file's appclip code to fit specific use cases.  Foreground or background color, or hide app developer logo.

This gives new customers the clear message that it is for app clips.  

AR anchoring => tracking position in the physical world.  Place guitar on clip.

[[Explore ARKit 5]]

Two ways to create.  Can use code generator, or download code directly from ASC.  Use CLI when testing or developing an app clip.  

Use ASC for registered URLs.  For best practices, keep app clip code flat.  Display code at upright position (not rotated)
Don't create codes that are too small, 1" minimum.
Ensure good visibility.  
Use clear messaging, e.g. call to action with english text.  Check HIG.
How to quickly generate appclip codes.

As mentioned, the easiest way to generate appclip code is to download from ASC.  For cases where you need more customization or need to script code generation, generate locally is a great option.

Generate appclip code using a template.  `AppClipCodeGenerator generate --type nfc --url https://fruta.example --index 4 --output fruta.svg`

SVG graphics are vector-based so they scale beautifully when you print them.  

Quick demo of appclip codes from generator.

# Wrap up
* Great app clips
* App clip card in safari and safariviewcontroller
* Test with local experiences
* App clip code

* https://developer.apple.com/documentation/swiftui/fruta_building_a_feature-rich_app_with_swiftui
* https://developer.apple.com/app-clips/



