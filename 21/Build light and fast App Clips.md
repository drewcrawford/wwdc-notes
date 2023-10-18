#appclips 

Small part of your app that's discoverable and downloaded at the moment it's needed.

Deeply integrated into the OS, integrated into safari, messages, QR code, NFC, etc.

# App clip size
Max size is limited.  10MB.  

Profiling workflow.

1.  Product=>Archive
2.  Distribute=>Development=>App clip.  Thinning=>all compatible device variants.  Rebuild from bitcode=YES
3.  Export to FS
4.  App thinning size report

Section fo reach device.  2nd-to-last line will specify the uncompressed size for each variant.

All variants are around the same size, so I'm not getting app thinning?

Open payload folder on FS, have multiple variants of iamges.  Save space by adding to asset catalog.

documentation zip and readme file that don't belong in here.

## Basic size optimizations
1.  Build settings
2.  Asset catalogs
3.  Inspect IPA
4.  Refactor

Apply to both app and appclip!

### Build settings
By default, Xcode uses "smallest, fastest".  But let's check.

Also check your archive config isn't "Debug"

### Asset catalogs

Most important step is to use asset catalogs. 

1.  Media is optimized as part of xcode build process.
2.  When customers download app/clip, product only contains assets for their device.

[[Optimizing storage in your app - 19]]
[[Optimizing app assets - 18]]

### Inspect IPA
I find it helpful to look in build phases to see every sourcefile.  If I see something that looks unnecessary, I remove and rebuild.

Localized strings files can be come inflated with duplicates and unused strings.  Delete things that aren't needed.

 Suppose as part of this process, you measure again and still aren't where you need to be.
 
 ## Advanced size optimizations
 
 ### Dependencies
 Only link with needed functionality.  Remember that smoothie account login framework?  Most of the time there's an apple framework that will help you accomplish your goal.
 
 ### optimize media
 
 Use PNG if you need a specific feature like transparency, or need higher quality.
 
 Consider PNG8 (8-bit?) for nonphotographic material.  Better than PNG(?)
 
 For photographic material, use JPEG and lean a bit on lossy compression.  Reduced filesize without an acceptable loss in quality.
 
 Video compression, audio compression.  Reduce bitrate, etc.
 
 For certain images, logos, icons, representing in SVG can lead to space savings.  Vector looks great in any size.
 
 SVG => Backs SFSymbols.  I highly recommend.
 
 ### Image variations
 
Instead of including several variations, include 1 base image and build variations at runtime.  Checkout Fruta sample.

Single image serves as the backing asset for 3 distinct uses.  

### Lazy loading

Ship lower-resolution placeholders with the appclip and use async image to replace them progressively after launch.

[[21/What's new in SwiftUI]]


# App clip invocation
## Two types of registration in ASC
* Default experience => specify metadata from safari, messages, etc.
* Advanced => QR scanning, app clips.

Registration takes time to propagate to devices.  

[[20/What's new in appstore connect]]
[[Configure and link your app clips]]

## Domain validation
URL viewed in safari or encoded in QR code must have a secure association via entitlements or file with app and app clip.

If not configured properly, your clip will not be surfaced.

[[What's new in Universal Links]]
[[Configure and link your app clips]]

Unfortunately, you just see your webpage.  What's the problem?  What steps can you take?

First, verify the syntax of your metatag and make sure it looks similar to this template.

```html
<meta name="apple-itunes-app" content="app-id=myAppStoreID, app-clip-bundle-id=appClipBundleID, app-clip-display=card">
```

example.com != www.example.com as far as domain validation is concerned.  Domain at the end of redirection should be the domain that serves the AASA file.

Safari won't display the full appclip card or smart banner if private browsing is enabled.

`awcutil dl -d checkcoverage.apple.com`

For more information,

[[20/What's new in appstore connect]]
[[What's new in Universal Links]]

If not fully configured, your website will be offered in Safari.  Usually because an advanced experience has not been created for the URL.  Even if your URL is the same as your website and your experience displays perfectly in safari, it's still necessary to create an advanced experience.

## Troubleshooting physical invocation

For codescanning, it's necessary to add the exact domain to your appclip's entitlements.  And serve the AASA file from this domain (as well).

Here we avoid the need to follow a redirection chain?  Due to UX performance concerns?

Developer settings => Clear experience cache




# Managing complexity
Each component should be indpdendent and directly launched into.  More suited to deeplinking, which is fundamental to appclips.

Checkout => directly invoked with a list of menu items.
By representing order as  a URL, you give your customers the opportunity to pay for your check at a table.

Customers can share their order with their friends.  With little overhead remaining, you can adopt appclips and increase reach.

Adds new lifecycle methods.  You might be tempted to place code that extracts URL, derive state and show some UI.

Before doing that, consider refactoring so that the code between your app and appclips are shared.  This avoids the need to maintain similar funcitonality in separate launch paths.

Consider creating a method like `respondTo(_ activity:NSUserActivity)` that can be called in both cases.  This reduces the overhead.

Since this is shared, you can make changes once and make benefits in app + appclip.

Any discussion on adopting modern features while maintaining quality would not be complete without mentioning testing.

For iterative development, set `_XCAppClipURL` to some URL, passed through to your clip.

To see in-development clip surface, see developer settings => Register local experience...

Otherwise, you're missing out on great opportunities to test how your clip is invoked.  Try out new features and see builds befor ethey're available in production.

[[21/What's new in App Clips]]

* Testing your App Clip's launch experience
# Unique functionality
Streamlined permissions
* Ephemeral notifications => 24 hours
* Location confirmation => confirm if they're in a vague area, without precise location information

Can provide pre-authorization on appclip card.  

Opt in via info plist. 

```swift
if let activationPayload = userActivity?.appClipActivationPayload {
  activationPayload.confirmAcquired(in: region)
  { inRegion, error in
    if let error = error as? APActivationPayloadError {
      if error.code ==
      APActivationPayloadError.disallowed {
        // User denied permission
        // Or invocation was not from visual code or NFC
      } else if error.code ==
      APActivationPayloadError.doesNotMatch {
        // Activation payload is not the most recent
        // Catch in testing. Handle as above.
      }
    } else if error == nil {
      // Platform was able to determine location
      // OK to check inRegion
    }
  }
}
```

Handle these states appropriately by checking the error.  Provide optiosn to resume transactions under more favorable conditions.

Most of you have enabled "Automatically manage signing".  If you're manually signing, make sure you obtain a recent profile.  When your customers upgrade from appclip to app, their data will be migrated faster.

Recent profile ensures that your app and clip have capabilities necessary for various improvements

# Wrap up
* Measure and reduce App Clip size
* Troubleshoot invocation points
* Modernize your project while maintaing quality
* Incorporate functionality unique to App Clips

[[21/What's new in App Clips]]

* https://developer.apple.com/documentation/Xcode/reducing-your-app-s-size
* https://developer.apple.com/documentation/app_clips/testing_your_app_clip_s_launch_experience




