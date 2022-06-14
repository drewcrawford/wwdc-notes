[[Customize your advanced Xcode Cloud workflows]]

# Integrate with Xcode Cloud
webhooks.
visibility into your builds.  Extract artifacts, etc.

Our integration:
Build starts.
Xcode cloud sends webhook
Process webhook
Call xcode API for build details
Create issue comment
Save comment in issue tracker


OpenAPI spec has all paths/models that make up our API.  Generate
swift code to talk to that particular path.  With strongly-typed object
you don't have to do any JSON encoding/decoding.

Client code is a swift package.  

App store connect => settings => webhooks.Xcode cloud knwos where to send webhooks.  Write code to process.

Code sample not provided.

## Recap
we have a good API.

# Manage your dependencies
Integrate with apple developer tools and services.  However, your xcodep roject may require additional dependencies or external tools to compile your code.

You can also make xcode cloud work with third-party dependency managers such as cocapods and carthage.

Refer to the xcode documentation for instructions on how to make dependencies available for xcode cloud.  

# Follow best practices
Refine your CI and CD practice to make sure always in a shippable state.
* SwiftLint
* Restrict editing
* Multiple start conditions

## SwiftLint
Open-source linter tool to enforce style guides
static code analysis

post-clone script
```bash
brew install swiftlint #environment includes homebrew!
swiftlint $CI_WORKSPACE
```

Temporarily deactivate this workflow to discuss with the rest of the team.  Come up with an agreement about coding styles and conventions and then decide which issues we want to fix.

## Restrict editing
Restrict who can make edits to workflow.  Evidently this is done in xcode.  Right click => restrict editing

It has a key symbol.  Means it is locked and can only be edited by the administrative users.  This means it has been locked by the administrator and cannot be edited by you.  makes it easy to manage access for complex workflows.

## Multiple start conditions
* same set of actions and post-actions
* Different build start conditions
* Easier maintenance
* Better manageability

Instead of creating 3 workflows, we can create a single workflow with all start conditions in one go.  Improves manageability and limiting the number of workflows we have to maintain.

> Fully-featured web-based experience

Everything I did in xcode can also be done on ASC.

# Wrap up
* use xcode cloud for development teams of any size to deliver high-quality apps for your users
[[Get the most out of Xcode Cloud]]
[[Customize your advanced Xcode cloud workflows]]

https://developer.apple.com/documentation/Xcode/Configuring-Webhooks-in-Xcode-Cloud
https://developer.apple.com/documentation/Xcode/Writing-Custom-Build-Scripts
https://developer.apple.com/documentation/Xcode/Configuring-Your-Xcode-Cloud-Workflow-s-Actions
https://developer.apple.com/documentation/Xcode/Configuring-Your-Xcode-Cloud-Workflow-s-Start-Condition
https://developer.apple.com/documentation/Xcode/Xcode-Cloud-Workflow-Reference
https://developer.apple.com/documentation/Xcode/Developing-a-Workflow-Strategy-for-Xcode-Cloud
https://developer.apple.com/documentation/Xcode/Developing-a-Workflow-Strategy-for-Xcode-Cloud
https://developer.apple.com/documentation/Xcode/About-Continuous-Integration-and-Delivery-with-Xcode-Cloud
https://developer.apple.com/documentation/Xcode/Xcode-Cloud
