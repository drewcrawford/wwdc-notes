#xcodecloud 
# Demo
When building PRs, xcode will test the merge together.

Can also narrow conditions to specific files/folders.
Cancel builds on push.

Depending on the needs of your team, therea re other condition types

# Start conditions
* Every change to a branch
	* Builds sourcebranch, ignoring PR
* Every change to Tag
* On a schedule
	* Great for long-running tasks that you want to run on occasion


# Set up environment
To choose your xcode/macOS version, just select from menu.  Can also set to "Latest Beta or Release".

Have the option to build incrementally.  However, you might want to perform a clean build instead.  Required to produce TF builds.

## Xcode Cloud API
* Access your data
* Configure workflows
* Kick off builds
* Run custom scripts

## env
Can mark secret.  See [[Customize your advanced Xcode Cloud workflows]] section.


# Configure actions
Define the work that you want the workflow to do.

* Build
* Analyze
* Test
* Archive

I need to select platform/scheme.  
Xcode cloud handles provisioning profiles and codesigning.  

[[Distributing apps in Xcode with Cloud Signing]]

## Test action
* Stable, reliable, reproducible environment
* Free up your local environment
* Run automatically

"Required to pass" -> overall build will fail if this fails.
Can "use scheme settings" or use a particular test plan.
Collection of simulators with different screen sizes.  Tests against xcode in the environment section.  But can also choose old simulators versions.

## Analyze action
Gives our users a stable and bug-free experience.  However we often forget to run it.  

## Build action
On occasion, you might need to simply build.
* Secondary build configuration
* Framework can be built on its own



# Configure post-actions
Can configure post-actions.  Run after all build, analyze, test, and archive actions complete.

* Send notifications
* Deploy with TestFlight

## notifications
* all build successes
* only fixes -> when a branch transitions from failing to passing
* don't notify

failures
* all build failures
* only breaks
* Don't notify

Integrates with Slack.  
By default, users receive notifications for builds they kick off.  But can add email here.

## deploy
Deploy to internal testers.  Quickly send builds to development team members.  

* Internal testers only
* Clean build not required
* Beta app review not required
* App store eligible not required

External testing
* deploys to external testers
* clean is quired
* beta app review required
* app store eligible required

### Internal testing
1.  Add an archive action
2.  Set archive deployment to TestFlight (internal only)
3.  Add a TestFlight internal testing post-action

### external testing
* Single branch
* select clean.  Guarantees your code will be built from scratch
* Set archive deployment to Testflight and App Store

Can also do stuff in ASC.  e.g., if you're away from Xcode.
# Recommended strategies

As you've seen, workflows have power/flexibility.  Can create as many workflows as needed to get your work done.

More workflows you can try out

## Release workflow
Enforce high-quality so users have a great experience
* Start condition: changes to `release` branch
* Execute full set of tests
* Run archive
* Deploy to external TF group

## Overnight testing workflow
* Runs on a schedule
* Runs on many simulators, multiple platforms
* Notifies the QA team on failure
* No deployment

I can add, edit, remove workflows as needed.

# Wrap up
* Workflows are flexible and extensible
* Run multiple actions across different platforms
* Automatically deploy with TestFlight

[[Meet Xcode Cloud]]
[[Customize your advanced Xcode Cloud workflows]]
