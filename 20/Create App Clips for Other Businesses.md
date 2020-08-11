#appclips 
# When to consider
* aggregates many businesses into a customer-facing app
* More users can discover these businesses through system-wide features
* Offers additional value for other businesses' existing customers
* Planning to consolidate multiple white-labeled template apps into one app

# Why create app clips?
* Use familiar technologies
* Re-use code
* Physical and digital invocation
* Brand identity shines through UI

|                          | One app clip experience for your business | Multiple app clip experiences for your own business | Multiple app clip experiences For other businessess |
|--------------------------|-------------------------------------------|-----------------------------------------------------|-----------------------------------------------------|
| Number of brands         | 1                                         | 1                                                   | Many                                                |
| Number of app clip cards | 1                                         | Many                                                | Many                                                |
| Suitable if full app...  | focused feature set                       | expansive feature set                               | aggregates multiple businesses                      |

# Getting started
* Get your existing app ready
* Build an app clip based on your app
* Submit your app with your app clip
* Create an advanced app clip experience for each business

## getting your app ready
* Make sure app handles each business you represent or promote
* Provide a custom experience that fits each category of business.  
* Provide the ability to browse, search, or explore your catalog
* Handle universal links

[[What's new in Universal Links]]

## build app clip 
* selectively include code, assets, and dependencies
	* [[Explore App Clips]]
* Set up associated domains, handle `NSUserActivity`
	* [[Configure and link your app clips]]
* Streamline transactions with ephemeral notification and location formation.
	* [[Streamline your app clip]]

### Additional considerations for URL routing

#### Launching for notifications

Fill out `targetContentIdentifier` payload in the notifcation payload.
iOS performs "longest prefix" match.  And can route to the correct app clip.
Please refer to
Documentation -> App Clips -> Enabling Notifications in App Clips
developer.apple.com/documentation/appclip/enabling_notifications_in_app_clips

#### invocations

Each invocation comes with an `NSUserActivity`.  Don't push or pop away if the URL corresponds to the same, inflight session.  This is because if the user is interrupted and backgrounds your app, we don't want to restart the session.

Save state before moving to a different session.  We don't support multiple windows.  That way, if the user goes back to the prior situation, we have their data

## Submit your app clip

Xcode automatically embeds the app clip in your app
Upload as a single submission

[[What's new in appstore connect]]



## create experiences

* Create an advanced app clip experience (in appstore connect)
* Specify the experience URL
* Choose to promote a different business
* send to Apple for review

### screencast

# app clip card
Invidiaully set up per business in ASC
* Header image
	*  use rich image or graphics, not text or ad
	* shown in card and loading screen
	* 3:2 aspect fill
	* Highest quality image available
	* 3000px x 2000px
	* apple will downscale
	* PNG or JPG
	* transparency is not supported; should be opaque
* display title
	* Your app is already attributed at the bottom
	* Titles represent the business, not your app
	* Localize if needed
* subtitle
* location association
* action
	* choose from a pre-defined list
	* automatically localized
	* send feedback for more actions

[[Design Great App Clips]]

# Types of icons
* app icon.  Shown in app clip card.  Shown in app banner.
* App clip icon
* app clip experience icon

## app clip icon
* represents your app clip, but not a specific app clip experience
* Shown across system UI similar to your app icon
* Treatment automatically applied.  Create square, opaque icons.

## app clip experience
"business icon"
* represents a particular app clip experience
* Associated with a URL
* Tapping launches app clip with the URL
* Visible across the system UI experience
	* app library
	* spotlight
	* proactive suggestions
	* messages, maps, safari
* Use a Maps POI icon for the business
* Business can upload an icon through Maps Connect
* If the business has no icon, iOS will call back to a generic category icon

# Wrap up





