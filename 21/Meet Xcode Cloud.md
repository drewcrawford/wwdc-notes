#xcodecloud

Introduce Xcode cloud, an easy-to-use CI and delivery service designed for developers just like you!

# Xcode cloud overview
1.  Push a change to your remote repository
2.  CI service - build, test, actions, 
3.  Artifacts, notifications

Workflow is a configuration that tells xcode cloud which actions to perform and when.

Not only available in xcode, but also ASC.  This includes starting/doing builds, managing workflows, artifacts, etc.

Xcode cloud is right nextdoor to testflight in ASC.

Personal notification settings, e.g. slack or email.
ASC provides fully-featured web-based experience.

> Your sourcecode is the heart of your project.  All aspects of Xcode cloud are designed to ensured that your dat ais protected

* Build environments are temporary
* Torn/down created from scratch between builds
* Source code isn't stored
* Data is encrypted and isolated
* You can delete your data at any time

# Set up your project
visit the Xcode Cloud section of the product menu and select "create workflow".
Select which app you'd like to onboard.

App begins with a default first workflow created.

Start condition, environment, actions, post-actions.

[[Explore Xcode Cloud Workflows]]

Authorize to acces sourcecode.  Primary repo, submodules, and private packages.  Publicly accessible packages don't require authorization.

It's important to note that auth process depends on source provider.  Granting sourcecode access is completed on the web.

First, I connect my apple ID with my source account.  Leverages provider's native authentication flow.

Install Xcode cloud application to my github org, enabling access to the repos I select.

In the final step, xcode cloud will register my application and bundle ID with ASC.  
# View results
Active and completed builds at a glance.  Opens overview page.

Complete build details.  Durations, environment confiurations, status, etc.

Rebuild and checkout buttons in top-right.

Logs page neatly organizes all tasks within the action, with filters to focus on areas that need attention.

Also have easy access to binaries, log files, etc.

This makes for a convenient way to access CI content for teams.

## Failure investigation

# Collaborate with your team

Appears that this mostly supports simulator.

Can run workflows without having to make code changes to start them.  Right-click on workflow.

# Wrap up
Your team's ability to iterate on both code and feedback productively is essential to creating a high-quality app

[[Explore Xcode Cloud Workflows]]
[[Customize your advanced Xcode Cloud workflows]]

