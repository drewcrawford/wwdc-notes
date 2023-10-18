#xcodecloud 
Learn how Xcode Cloud can help teams of all shapes and sizes in their development process. We'll share different ways to configure actions to help you create simple yet powerful workflows, and show you how to extend Xcode Cloud when you integrate with additional tools.

# Solo developer
one-person show.  Simplicitly, rely on, maintain.

Xcode cloud, the entire process of pushing, building, and distributing can be achieved in one workflow.  

Demo.



# Medium-sized team
* pull request workflow
* beta build workflow
* release workflow

```bash
#!/bin/sh
# ci_pre_xcodebuild.sh
#

if [[ "$CI_XCODEBUILD_ACTION" == "archive" && "$CI_WORKFLOW" == "Beta" ]]; then
    echo "Replacing app icon with beta icon"
    mv BetaAppIcon.appiconset ../App/Assets.xcassets/AppIcon.appiconset
fi
```
See [[Customize your advanced Xcode cloud workflows]]


# Large team


## Release workflow
Create a workflow that archives and uploads a version of the app.

Just like the other examples.
## Run tests reliably

Consider 'not required to pass'?  Result of action won't be required.

Useful because it means you can continually run your tests off of the critical path.  Aggregate data to assess how tests are performing and if they are reliable.

[[Author fast and reliable tests for Xcode Cloud]]
see docs



* https://developer.apple.com/documentation/appstoreconnectapi/xcode_cloud_workflows_and_builds
* https://developer.apple.com/documentation/Xcode/organizing-tests-to-improve-feedback
* https://developer.apple.com/documentation/Xcode/Making-Dependencies-Available-to-Xcode-Cloud
* https://developer.apple.com/documentation/Xcode/Configuring-Webhooks-in-Xcode-Cloud
* https://developer.apple.com/documentation/Xcode/Environment-Variable-Reference
* https://developer.apple.com/documentation/Xcode/Writing-Custom-Build-Scripts
* https://developer.apple.com/documentation/Xcode/Xcode-Cloud-Workflow-Reference
* https://developer.apple.com/documentation/Xcode/Developing-a-Workflow-Strategy-for-Xcode-Cloud
* https://developer.apple.com/documentation/Xcode/Configuring-Your-First-Xcode-Cloud-Workflow
