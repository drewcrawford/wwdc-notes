At wwdc 2020, we announced privacy nutrition labels to privde peopel with an easily-glanceable and understandable summary of how apps collect and use data.

Nutrition labels are one of the many ways you can communicate privacy practices.

Labels can help you explaint o people how you use the data your app collects.  Sharing a label is a requirement for all appsin the app store.

* used to track you
* linked to you
* not linked to you

Alternatively, if data is not collected, an alternate language is shown.

More detailed version in the app store.

# Label creation process
* Create inventory
* Enter responses into ASC
* Update repsonses if needed

How to craft inventory for your app.

Collection categories
use cases
linked to identity
tracking

You might not be thinking about data categories in your app.  Where to start?

To a feature inventory
consider what data powers each feature
Find a framework to document this info that works for your app.

When creating this inventory, a number of resources
* consult app stakeholders
* use app privacy report
* review data stored server-side

Keep in mind that while a network audit may be helpful, it is not comprehensive and you should use it in combination with other strategies.  Also audit any data that you retain on a server.

## Partners and SDKs
You're responsive for your whole app, including SDKs
Review SDK documentation
Some provide specific guidance for nutrition labels which you can use to ensure your label is comprehensive.

## Data minimization
Identify opportunities to make changes
* minimizing data collection
* on-device processing
* storing data not linked to identity
[[22/What's new in privacy]]

## Enter responses into ASC
From your app's page, open the app privacy section.  Asked whether you collect data.  DAta is considered collected when it is transmitted off the device.  Server logs, user profile, or analytics.

## Collection
Declare ocllection for all features of the app
Labels supplement a privacy policy and other dislcosures or consent
> We recommend working with legal counsel to evaluate those requirements

Email address, phone number, and payment info.  Provide more detail for each data category.

For each category, be asked to declare what use cases the application supports.  Be asked to disclose whether data is linked to a user's identity.  Data is considered linked if it associated with an account, device, or user profile.

Evaluate whether you need to link this, or you can avoid it.

Identify if used for tracking devices
* linking data from your app about an end user or device, user identifier, device identifier, profile, to third party data used for advertising purposes
* sharing data from your app about particular user or device with a data broker.

Nutrition labels are intended to reflect all data you colelct, even with user permission
Disclose data used for tracking on the nutrition label

Use ATT to request permission to track

[[Explore App Tracking Transparency]]

Once you've submitted this information..

## Update if needed
Re-evaluate your label when releasing new features and on an ongoing basis.  Adding new features, implementing new or updated SDKs, or using already-collected data in new ways, evaluate whether any changes are needed.

# Definitions and examples

All of this information is available in docs.  Today, I'll be highlighting a few examples based on our experience, as well as questions from developers.

How to disclose use of IP addresses?
* identifier
* appropximate location
Our guidance is to *declare categories based on use*.
If using to locate the user, declare location.

## Product interaction
Interactions inside the app, which screens people open.

## Browsing history
Interactions with content *not* part of the app, such as an in-app browser.

## Search history
Any searches in the app, for any content.  Both in the app, or in-app browsers.

## Optional disclosure
Certain collection is *optional* to disclose.

* Infrequent, optional, and independent from the app's primary functioanlity, clearly discloses all collection at submission time, and has limtied use purposes
* ex, not used for tracking or advertising
Feedback form snad report problem forms may qualify
Full details in developer documentation

# Wrap up
* Reach out to stakeholders
* Include SDK functionality
* Tracking requires user permission
* Review label when making changes



https://developer.apple.com/app-store/user-privacy-and-data-use/
https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy