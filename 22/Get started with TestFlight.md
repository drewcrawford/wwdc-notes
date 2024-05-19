Discover how you can use TestFlight to improve your app experience and ready it for release on the App Store. We'll take you through an overview of TestFlight, including how to invite testers and provide information to them about testing. We'll also provide best practices for receiving feedback and explore how you can get the most out of the testing process.

# TestFlight overview

Essential to creating high-quality app experiences.  Deliver great apps and experiences to your customers on the app store.

App store is available in over 170+ regions on more than 1.5B devices.
40+ languages

A ton of different people with different devices, language, AX needs.  By testing your app, you can make sure you're providing a great experience for app store users around the world.

## testing tools

Xcode
xcode cloud
Manually test your code with different devices/oses with simulator
Use instruments to performance test.

## testflight

gather feedback from real people in a privacy-friendly way.  
Get feedback before your app is live, use this info to update your app accordingly.  TF is included as part of your apple developer program membership, one of the most popular tools that we ovver.  Distribute your app across all apple platforms.

1.  Upload build
2. add testers
3. get feedback
4. publish on app store

Let's walk through an example.
# Using testflight

To best illustrate this, let's imagine we're preparing a new food truck app.

First select the architecture we want to build, 'any iOS device'.  Archive.

Distribute app from organizer.

select 'app store connect' as distribution method.  
Select 'upload' to automatically send build to ASC.  Before it can be uploaded, click next to let xcode create an app record for us.

Include symbols.  Manage version/ build number, so distribution assistant can detect whether we have a valid build number.  Invalid number, assistant will give us the options to atuomatically increment to a valid number.

Once our app is uploaded, it's ready to be tested with test flight.

## Best practices

* select 'app store connect' for distribution
* be mindful of version and build number.  When uploading a new build, have to specify a build string with a number greater than previous.
* Use xcode auto signing and version management.

## back on topic
using ASC to find testflight tab.

Simply upload an additional build.  Details of our newly uploaded build, including status and when it expires.  Adding test details helps our testers know what to pay attention to when they test our app.

Let's add some in for this build.  This is our first time releasing this app.  Change design of the foodtruck, mention in test details as well.

Filling out test details specific to your build is the first thing testers see when they go to test your app.

## best practices

* update on a per-build basis
* Keep it simple: use short sentences or bullets
* Call out features to test and known issues

## adding test information
Test information in lefthand organizer.
Need to add our beta app description and an email address.  App description is visible to tester from TF app.  Feedback email address we entered here.

We will need to fill out beta app review information before we can distribute our app  to external testers.  Only have to update once per app, and again if info changes.

With our build uploaded and configured, now we can add testers.  Add two types of testers: internal, external.  Since our app is still fairly new, let's get feedback from a few people within our team.  Add to our app as internal testers.

## Internal testers
Members of ASC team
Up to 100 internal testers per app
Each tester, up to 30 devices
(can) Automatically distribute all new builds to internal testers

Plus button next to internal testing to create a new group.  Groups are a powerful tool because they allow us to create repeatable testing processes.  Enable automatic distribution checked.

After i add testers, they'll receive an email invite to test our app.  Email includes a link to view our app in testflight.

## External testers
Up to 10k testers per app
Invite via email invitation or public link.

Submit build to app review.
Test in-app purchases without incurring charges.  Improve these experiences before going live on the store.

Like with internal testing, click on blue plus button next to internal testing section.  Let's invite some chefs to test our app.  Let's call this group 'chef testers'.

Manually add builds.
Email invitation:
* tester email required
* sent via email
* feedback linked to tester
* can re-invite to future apps or beta
public link:
* tester email not required
* set enrollment limit for link
* multiple distribution options with link
* may share feedback anonymously

Always good to start with a smaller group of customers and then expand.  Can edit limits later!
Feedback provided by external testers appears in the same place as internal tester feedback, etc.

Our build is already uploaded to ASC.  Just takes us a few steps to submit for publishing.
Since we've already uploaded our build to TF, we won't need to upload again.  Select build from dropdown when prompted.  Submit for review, that's it!

As you consider incorporating testflight into your release process...

# Xcode Cloud
CI and delivery service built into xcode and designed expressly for apple developers.  Accelerates delivery of hq apps by bringing together cloud-based tools
* run automated tests in parallel
* easily deliver apps to testers

works with workflows

[[Get the most out of Xcode Cloud]]
[[Meet Xcode Cloud]]
[[Explore Xcode Cloud Workflows]]

# Best practices
* start small, then expand.  Consider adding testers over time.
* Know what you want to accomplish with each build/test
* Use groups to create repeatable testing processes
* Use xcode cloud to automate your testflight workflow

hope that helps!
# Resources

* https://developer.apple.com/testflight/
* https://help.apple.com/app-store-connect/#/deva50a6ab41
* https://testflight.apple.com/
