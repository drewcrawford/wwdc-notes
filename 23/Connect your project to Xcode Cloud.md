Unlock the benefits of continuous integration and delivery in Xcode Cloud with source code management tools. Learn how to set up Xcode Cloud with a self-hosted source control management provider like GitHub Enterprise, troubleshoot common issues, and explore key project maintenance tips.

# Introduction to Source Control
levels up your development practices!  Accelerate delivery of high-quality apps, etc.

Tracks changes over time.  

SCM providers.  BitBucket, GitHub, GitLab.

[[Meet Xcode Cloud]]


# Xcode Cloud source code flow
Data is encrypted, in transit and at rest.  Industry-standard, strong encryption protocols.  Build artifacts.  

Ephemeral build environments.  Strong, secure isolation boundaries.
we use HTTPS connections to communicate with your scm providers.

xcode cloud creates a temporary environment.  code is pulled down.  Performs a git clone operation, using remote repo.

[[Create practical workflows in Xcode Cloud]]



# connecting Xcode Cloud to GitHub Enterprise
grant access to SCM provider.  Self-hosted projects need more information.

GitHub enterprise.  Host the backyard birds app.  Get started with onboarding by clicking on 'get started' in the cloud tab.  Start onboarding dialog, where you'll be asked to choose app or framework to connect.  

xcode cloud connects from specific IPs
17.58.0.0/18
17.58.192.0/18
57.103.0.0/22

"Requirements for using Xcode Cloud" docs


# Common project maintenance
Won't have to go through these steps again.  As your project grows and evolves, change where source is hosted.  Keep xcode cloud connected.

If your whole project has moved to an SCM provider, update primary repository in ASC.

Point single workflow to new repository.  Under primary repository, change for a specific workflow.

# Wrap up
* xcode cloud and source control work together
* cloud and self-hosted repositories on BitBucket, GH, GL.



# Resources
* https://developer.apple.com/documentation/Xcode/Requirements-for-Using-Xcode-Cloud
* https://developer.apple.com/documentation/Xcode/Xcode-Cloud