#design  #ios 

Great navigation is often unnoticed.  navigation involves teaching people about 

* how things behave
* where things belong
* how things work

The goal of navigation is to provide a foundation of familiarity.

When it deviates too far from expectation, we often feel frustrated and sense that an app is hard to use.

# Tab bars
* use tabs to reflect your information hierarchy
* translate an already clear grouping between different app areas.
* top-level content.  
	* each tab communicates a menu of options.

In practice, and for various reasons, ti can be easy to lose sight of.

## **Balance features throughout tabs**.

Map view,
upcoming itinerary
popular routes

It can be tempting to add all functionality to first tab because it's available at a glance.
Or maybe you've added stuff over time.

Today I invite you to reconsider this.   People may scroll a lot, parse unrelated features, etc.  Filtering a map view and editing an itinerary are two different features and mindsets.

Be cautious of combining your app's functionality.  Much easier to understand an app's functionality when it's well-organized and clearly communicated.

*Why do people use your app*?

A few things really well.  

Let's re-assess the tab bar to understand how a route may be used and how that content may be represented int he app.

* Route details
	* Place
* City details
	* list of routes
* Places
	* top level hierarchy
	* great justification for a tab bar item.

Look for these natural ways to develop relationships.  

Even if people are unfamiliar with the content of your app, and especially if they are.  Best to communicate functioanlity and content clearly.  Wehre it belongs in hierarchy.

Makes navigation intuitive.

## Avoid duplicating functionality into a single tab
In content-rich apps, a tab called "home" may seem attractive.    Maybe people don't know some functionality exists, so encourage engagement by repeating functionality in some obvious place.  Out of fear that some functionality won't be discovered.

This isn't about duplicating **content**.  It may make sense to have similar types of content exist on many tabs but *organized* in a different way.  We're talking about *functionality*, e.g. *actions*.  The redendancy creates confusion.

Home tabs disrupts hierarchy.  If functionalities are duplicated and added toa single screen without context, it creates redundancy and confusion.

Home => Every feature is fighting for real estate.  It creates a diassociation between content and understanding how to act on it.  Remove the home tab altogether, redundancy prohibits people from understanding where content belongs and why.

Repeated functionality may cause someone to tab jump, becuase the action exists in another tab too.

Never force someone to change tabs automatically.

## Keep tabs persistent throughout your app
Toggle between diffeernt levels of hierarchy while maintaining context in each.
* places => look at new route
* itinerary => compare to itinerary

This only works well if tabs have defined purpose and represents specific categories of content.

## Use clear and concise labels
You land on the middle tab after launch.  OPther tabs are specific, easy to understand, and immediate sense of what the app does and how to use.  The labels are representative of the content.  They all represent core functionality with a succinct label.

Powerful control for navigation.

* use tabs to reflect information iherarchy
* balance features throughout tabs
* avoid udplicationg functionality into a single tab
* keep tabs persistent throughout your app
* use clear and concise labels

# Hierarchical navigation
Push.  Such as pushing in to more detail.
Or, with a modal presentation.  These are nonintrusive and familiar ways.

Expected default when drilling further into hierarchy.  Directly reflects hierarchy, literal representation of drilling into more detail.

Modal => self-contained task and interface.  Workflows that are independent.  Someone has enough information in that view to make decisions and complete a task.  Create focus by separating people from the information hierarchy.

## To traverse your app's hierarchy (push transition)
Top-level and secondary content.  I access the supplemental views by drilling into the hierarchy.  

Content should become increasingly more specific and there should be fewer options as I push in.

## Keep tab bar anchored to the bottom of the screen
Gives access to core areas of your app because it's always available.  Explore content at different hierarchies.

Swipe left to right without losing hierarchies in other tabs

## Use the navigation bar to orient people
I never have to remember whree I came from, text on back button.

## Use with a disclosure indicator
aka *chevron*.  Points in the direction you're expected to transition to.
Disconnect between what the UI represents and what the interaction that follows.

In western cultures, we read LTR.  If your app supports RTL, transition of pushing is flipped to create an association with the flow of content.

## When navigating frequently between content
Messages app.  Even if the hierarchy is relatively flat, I can move in and out of messages.  If meach message was a modal, it would be difficult to seamlessly move between different chats.

Dismissing a modal whe it's not relevant makes people ahve to think about leaving the screen.  Pushing allows frictionless transition.


* traverse your app's hierarchy
* keep the tab bar anchored to the bottom of the screen
* use the navigation bar to orient people (title, back label)
* use with a diclosure indicator
* when navigating frequently between content


# Modal presentations
Isolating someone into a focused workflow or self-contianed task

## Present from the bottom of the screen
Interrupts information hierarchy.  It covers your tab bar.  Prevents peopel from drilling into your app, intentional disruption.  Reinforces focus.

**What is a self-contained task?**

## Use for a simple task, multi-step, or full-screen
Simple task => creating a reminder.  Edit and modify the entry fields.  achieve without distraction, minimizes the possibilty of abandoning the flow.

Multi-step task => adding a transit card to wallet.  It may seem counterintuitive, but remember.  The purpose is to reinforce focus by hiding the tab bar and preventing peopel from moving around the app before task is complete.

Article, video, full-screen content => starting a workout.  Great scenario for modal presentation.

## Dismiss a modal with 'cancel' and preferred actions
Think about the navbar as a guide for wayfindings.  Left/right actions, etc.

Right label => preferred action.  Done, next, send, submit.
Dismisses the modal, previous state of the app should be preserved.
Having the action be inactive is a good way to clarify that fields are required.

Dismiss => cancel.  Clearly indicates that I"m abandoning the workflow.  If I've entered
information, maybe display an alert?  Action sheet, etc.  If I haven't interacted with the UI, tapping cancel should just dismiss.

## Use close for minimal interaction
Close symbol works here because there's no user input.

Without a clearly labeled action, people will wonder what hapepns if I tap close.  Using labels in the navigation bar is preferred.

## Limit modals over modals
Tint color.  Indicates actions are tappable.  

Try to limit multiple modality stacks but sometimes they're required.

* present from the bottom
* use for a simpel task, multi-step, or full-screen
* use preferred and 'cancel' actions appropriately
* use close for minimal interaction
* limit modals over modals

hope that helps

* https://developer.apple.com/forums/tags/wwdc2022-10001
* https://developer.apple.com/forums/create/question?&tag1=137&tag2=289&tag3=300&tag3=342030
* https://developer.apple.com/design/human-interface-guidelines/
