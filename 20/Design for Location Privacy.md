#design 
#location 
#privacy

People share various data.
In iOS 14, people can share approximate location instead.
This *blew our minds*.  Our team designed apple maps around precise location.  It's the focal point, it's used throughout maps, and navigation depends on it.

Weather, wallet, photos, etc.  

Hard for us to imagine how our apps could work without precise location data.  If you're feeling similarly, think about how to better respect location data and privacy.  Hopefully our experience will inform your work.

[[Build Trust Through Better Privacy]]
[[What's new in core location]]

* Prioritize user control.  
* Build trust through transparency
* Offer proportional value

Biggest change is reflecting approximate location on the map.  We needed a different annotation to represent approximate data accurately.  We designed a shaded circular area to indicate approximate location.

App audited to determine where else we use precise location data.  Calculating arrival times is not feasible with approximate location, so we removed it.

# Prioritize user control
* Don't require precise location.
	* allow people to do as much as they can without this
* replace precise data with approximate where possible
* remove non-essential use of precise location

# Build trust through transparency
In maps, we invest in writing.  Clear language explains what we ask for and what value it provides.  Clarity and accuracy gives peopel the right knowledge.

Communicate to people the result of their choices.  If they choose to share approximate location, we added a status bar that indicates "Precise location: Off".  This appears larger/stronger and gives them access to control the setting.

Once people pan around, the visual treatment reduces in prominence.  So it doesn't distract from other functionality.

* Invest in writing and communication
* make status easy to access
* Allow users to change their mind


# Offer proportional value
Some features require precise data.  Limit requests to specific features that provide it and connect to value in the UX.

e.g., turn-by-turn directions.  So when peopel indicate they want directions, we ask for *one-time access* to precise location.

We try to ask people as close to giving them directions as possible.  So people sharing approximate location will still see all the "get directions" buttons.  When they tap it, they can input their places of origin manually and destination.  Since maps needs precise location to auto-populate place of origin, we added a new option as a search result, allowing the user to choose that option.

This gives people the option to create a route from their precise location.  When someone chooses this, then we give them the precise data.  Connected to user taking action, e.g., asking directions, and positions close to when they will receive direction in time.

* Only ask for precise location when you really need it
* Request precise location in response to user action
* Position the request close to the value it provides


# Wrap up
* prioritize user control
* build trust through transparency
* offer proportional value

lamb