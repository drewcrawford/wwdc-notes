#safari 

New web extentions API.

[[Meet Safari Web Extensions on iOS]]
[[Meet Safari Web Extensions]]

# Non-persistent background page
Using JS, HTML, and CSS.  Background page.

No visible UI but it can react to events.  Persistent pages do not close.  Like invisible tabs that a user can never close.

Performance matters.  So we have a non-persistent page.

**Required for iOS**

* Event driven
* Unloaded and loaded as needed

`"persistent":false`

* Use `browser.storage` to write info to disk
* Register listeners at top level
* Use `browser.alarms`, not timers
	* Timer won't be invoked if bg page has unloaded
* Remove calls to `getBackgroundPage()`
* Don't use webRequest.  Too high-frequency, unclear if it's supported or not

## Demo
See process in safari running forever.
Make persistent instead
Now process exits
Use browser.storage to persist state
Need `storage` permission in manifest

# Declarative content blocking
* Expressed in JSON
* Grouped into rulesets
* Enables cross-platform content blocking

1.  Specify a ruleset
2.  `declarativeNetRequest` permission
3.  rule
	1.  id
	2.  priority
	3.  action (block,allow,upgrade sceheme)
	4.  condition
		1.  regexFilter => matched against resource
		2.  resourceTypes => e.g. `script`  Various options here.
		3.  `exludedResourceTypes`
		4.  `domainType` e.g. thirdParty
		5.  `isUrlFilterCaseSensitive`
	
## Demo
	
Look for errors in extension preferences

# Customizing new tabs
Personalization
New tab override

* Declared in manifest
* Controlled by user
* `browser_url_overrides: {}`

## Demo

# Wrap up
* Non-persistent background page
* Declarative net request
* New tab override
* Download sample projects
* Provide feedback

[[Meet Safari Web Extensions on iOS]]
[[Design for Safari 15]]

* https://developer.apple.com/safari/download/
* https://developer.apple.com/documentation/safari-release-notes
* https://developer.apple.com/bug-reporting/

