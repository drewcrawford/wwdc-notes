
#appstore 
Discover the latest updates to App Store Connect, the suite of tools used to manage and submit apps to the App Store. Explore how you can use the latest features to test, price, promote, and automate the management of your app more easily. We'll also share enhancements to tools like TestFlight and the App Store Connect API.

# Monetize your app
Features to help you run your business on the appstore.

StoreKit for SwiftUI.
simplest implementation, customize, etc.

If you promote IAPs on the appstore, can use the appstore promotion image in the product view.  
If you offer IAPs or offer a paid app, considerpricing.
We're updating pricing capabilities.  Choose from more price points.
Select a base region.
Manage international pricing
Set availability of in-app purchases and subscriptions
[[Meet StoreKit for SwiftUI]]
[[What's new in App Store pricing]]


# Manage testers
Start thinking about who your testers will be, how to test changes, etc.  Test your prerelease builds on iOS, watchOS, tvOS, macOS, and now xrOS.

making it easier to manage testers, builds, and test new usecases.  Data about testers to help understand how engaged they are. 
* status -> invited, accepted, isntalled, etc.
* sessions
* crashes
* feedback
* additional column for devices.  Most recent only?
* add filter to view/manage segments of testers
* bulk-select to resend invitations, add to group, delete, etc.
* available via API

when distributing a build from xcode, you may have a prototype build.  For this, we're adding a testflight internal-only selection to the distribution workflow.
ensures these builds can't be submitted for appstore review.
clearly marked in appstore connect.

ability to upload testflight 'what to test'.

## include notes for testers
add a plaintext file to TestFlight folder
Pull commit messages in a build script

passed to ASC and will be distributed to your testers when your'e ready to test your build.

Share your subscriptions and in-app purchases with group.

combine sandbox test accounts into a group.  This is under "users and access" "family sharing".

## sandbox on device.
* view family group
* modify renewal rate
* test interrupted purchases
* clear purchase history

[[Simplify distribution in Xcode and Xcode Cloud]]
[[Explore testing in in-app purchases]]


# Build your store presence

appstore privacy nutrition labels summarize data practices in a simple, easy-to-read label.  When answering app privacy questions in asc, you need to indicate the types of data you collect from customers.  with xros, we're adding a few new datatypes
* environment scanning
* hands
* head

especially relevant for xrOS apps but could apply in other platforms.  

## pre-orders
flexibility to use pre-orders on a regional basis.  Launch your app to a limited set of regions (soft-launch) and then offer your app for preorder in other regions.

Redesigned availability page to manage app state across regions.  

product page optimization.  Gives you insight into which product pages your users like best.  Making a change so tests continue to run until you choose to stop them and won't be affected by new versions.  View/monitor currently-running tests.  Keep in mind that any changes to a product page could impact results of a running test.

[[Explore App Store Connect for spatial computing]]
[[23/What's new in Privacy|What's new in Privacy]]
[[What's new in App Store pre-orders]]

# Automate with APIs

API collections automate many areas of the appstore.  This year, we've launched IAP/subscriptions, customer reviews and responses, and sandbox testers.

Adding support for gamecenter.  Easier and faster to set up gamecenter features and build consistent experiences across all aspects of your game.

* configure and archive leaderboards/achievements
* submit scores and achievements
* Remove scores and players
* Match players using custom rules

* Generate marketing and customer service API keys
* create a user-based key

# Wrap-up
* try out new capabilities
* contact us for unlimited 24/7 support
* Submit feedback
