#xcodecloud 

Discover how to share your app using Xcode's streamlined distribution, which allows you to submit your app to TestFlight or the App Store with one click. We'll also show you how to use Xcode Cloud to simplify your distribution process by automatically including notes for testers in TestFlight, and use post-action to automatically notarize your Mac apps.

Quickly/easily build your app, distribute, test, get feedback, refine, etc.

Fortunately we have the right tools for the job.  

testflight, appstore, notarization.

* xcode organizer window
* xcode cloud workflows

# Express TestFlight distribution
what is an archive?
record of the app build
contains an optimized release build
contains debug symbols (.dSYM)
Contents repackaged when uploading
Product->Archive.
Currently, I have the simulator selected.  Xcode is smart and will switch to iOS device.

Distribute app.  New in xcode 15, you can choose from 1 of several streamlined options.  Upload or export your app easily.

1.  Upload for TestFlight and app store
2. Internal testflight -> share with your team.  Prevented from app store submission, use with development branches
3. debugging -> optimized release build, sign with development certificate, isntall on team's registered devices
4. release testing -> optimized build, sign with distribution, install on team's registered devices

each of these use recommended settings
* uses automatic signing
* includes symbols info for crash reports
* auto-increments build number
* strips swift symbols

choose any export or upload option
choose default options and more
see stuff in feedback tab.  

## adding testflight support.
Use the menu integrate->manage workflow
edit archive action to add tf (internal only)
add a testflight internal testing post-action to add a testflight group.

check out developer docs for "including notes for testers with a beta release of your app".  This features the build scripts I'm using?

I guess we're pulling this info from git.

```bash
#!/bin/zsh
#  ci_post_xcodebuild.sh

if [[ -d "$CI_APP_STORE_SIGNED_APP_PATH" ]]; then
  TESTFLIGHT_DIR_PATH=../TestFlight
  mkdir $TESTFLIGHT_DIR_PATH
  git log -1 --pretty=format:"%s" >! $TESTFLIGHT_DIR_PATH/WhatToTest.en-US.txt
fi
```

With the help of xcode cloud,a ny time I push changes, my team receives a new build, etc.  I can now continuously integrate feedback, deploy improvements, etc.  Freed up to focus on development and ship the best possible result.

# Automating notarization

overview of notarization.

See [[all about notarization]] and [[What's new in notarization for Mac apps]]

1.  Produce an archive.
2. distribute app
3. direct distribution

This year we added upport in xocde cloud.
Setup a workflow with the new notarize post-action
Xcode cloud automatically starts a build based on the configured start conditions.
Only admins and app managers can set up workflows, etc.

Post-action: notarize.  that's it.  

# Wrap up
* Use xcode streamlined distribution for quick uploading
* Use TF internal distribution to iterate on changes
* Use Xcode Cloud workflows to automate distribution, including notarizing mac apps.
* 




# Resources
* https://developer.apple.com/documentation/Xcode/distributing-your-app-for-beta-testing-and-releases
* https://developer.apple.com/documentation/Xcode/including-notes-for-testers-with-a-beta-release-of-your-app
