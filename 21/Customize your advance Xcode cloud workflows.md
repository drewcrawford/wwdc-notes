#xcodecloud 
Continuous integration and delivery service.  We've shown how you can set up end-to-end workflows to test and distribute your app.

  Integrates with essential day-to-day developer tools.  You may have in-house or proprietary tools that are a key part of your pipeline.  
  
  How to customize 
  
  # Environment variables
  e.g., use a staging environment vs production?
  Simple `KEY=VALUE` pairs
  Define information that controls behavior of a build
  
  ## Secret environment variable
  Keep sensitive information secret
  Secret variables are encrypted and redacted from logs
  
  Value of the environment variable is hidden from view and will be stored securely, can no longer be viewed in workflow editor.
  
  # Custom scripts
  Lots of flexibility in how you set up actions to run in workflows.  But sometimes you want to run custom logic or additional commands.
  
  * Allow running custom commands during an action

Three types
* Post-clone
* Pre-xcodebuild
* Post-xcodebuild

Each time xcode cloud runs an action it performs a series of steps.  

1.  Setup environment and clone source
2.  Post-clone
3.  Resolve other dependencies
4.  Pre-xcodebuild
5.  Xcodebuild action
6.  Post-xcodebuild
7.  Save artifacts

Can put this in `ci_scripts` folder
When xcode runs your actions, it looks for each script.  Can run on branch, etc.  Don't need to configure your workflow, if they're there, they'll be run.

Note that the name of `ci_scripts` folder contents must exactly match

* `ci_post_clone.sh`
* `ci_post_xcodebuild.sh`
* `ci_pre_xcodebuild.sh`

Various environment variables are provided, long list @ 6:09

* `CI_PRODUCT_PLATFORM` -> iOS, macOS, etc.
* `CI_WORKFLOW` `CI_XCODEBUILD_ACTION`, can check workflow and if it's the archive action for example

[[Explore Xcode Cloud Workflows]]

Example script to replace icons with beta
```swift
#!/bin/sh

#  ci_pre_xcodebuild.sh
#  Fruta
#
#  Made in Vancouver, Canada
#  

# is this a pull request or not?
if [[ -n $CI_PULL_REQUEST_NUMBER && $CI_XCODEBUILD_ACTION = 'archive' ]];
then
    echo "Setting Fruta Beta App Icon"
    APP_ICON_PATH=$CI_WORKSPACE/Shared/Assets.xcassets/AppIcon.appiconset
    
    # Remove existing App Icon
    rm -rf $APP_ICON_PATH
    
    # Replace with Fruta Beta App Icon
    mv "$CI_WORKSPACE/ci_scripts/AppIcon-Beta.appiconset" $APP_ICON_PATH
fi
```

Note that I'm using `$CI_WORKSPACE` to construct path.

Now open a PR with these changes, and xcode cloud does the stuff.

* Logs from custom scripts are available in the Activity Log and in the Logs artifact (stdout and stderr).
* Add logging and resiliency
* e.x., retry requests
* Custom scripts exit codes are respected
* Source code is not available when running tests.  This happens in a different environment than the build.
* Therefore, post-clone script won't run in these environments.  So shell scripts, small tools, etc., must be entirely contained within the `ci_scripts` folder.

  # Additional repositories
  Many projects are built using tools, libraries, and frameworks.  git repositories, shared across projects, etc.
  
  Demo involving SwiftPM.  Xcode cloud provides an easy UI to address the repository issue.  Evidently you can even resolve this from ASC.
  
  * Simple UI to grant additional repositories
  * Valid for any git operations (including cloning a repository inside a custom script or referencing a git submodule)
  * Valid for all dependency management tools
  
  [[Discover and curate Swift packages using collections]]
  
  # Webhooks
  Sometimes, you and your team may want to collaborate.  e.g, notify testers when a build is ready.
  
  * Allow further cusotmization of your workflow
  * Send messages to an endpoint of your choice
  * Sent at different moments of a build's lifecycle

* create build
* start build
* complete build (succeeded or failed)

Adding webhooks is easy (ASC).  Settings, webhooks, plus.

Payload is JSON blob with info about build and product 

`app` name
`workflow`
`ciProduct`
`buildRun`
etc.

Need to setup an app or service to get payload etc.  Sample call of how to achieve using Swift on AWS Lambda.

Pretty much decode to JSON etc.

[[Use Swift on AWS Lambda with Xcode]]

ASC can show you request/response for debugging webhooks.

Examples

* Create issues in bug tracker for issues
* Send notifications to paging system when build fails
* Initiate downstream build as part of release workflow

# Wrap up
* Environment varaibles
* Custom scripts
* Additional repositories
* webhooks

