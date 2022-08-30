Share the exciting new opoprtunitie sto create extensions for safari web inspector.

maybe you're debugging a JS library, need something more specific to debug?  Personal workflow scenarios.

Add your own tasks in web inspector in safari.

# demo
NOt only can I see builtin tabs, but also the tab for the extension I enabled.  Because we've just enabled it, we first have to give it permission to work with the current page.

Same duration option as other extensions.  For now, let's give it access for one day.

Like other safari web extensions, they're distributed as an app in the app store.  To build your own extension, you need xcode.

Project templates to help new safari extension app.

`xcrun sfari-web-extension-converter` to convert an existing web extension from another browser.

[[Meet Safari Web Extensions]]
and docs

# Basic structure
In a typical safari web extension, we start with a manifest file.
Declares a background page with behind-the-scenes logic.
Content scripts.

DevTools background page.  This has access ot the unique devtools API.  Limited set of content script APIs.  Create tab pages taht will be shown in web inspector.

If you have multiple web inspectors, each has its own instance of the background page.  Staying alive for the duration the inspector is open.

Note that you need to pick a team and bundle ID based on apple developer account your'e using.  

Delete popups and stuff we don't need.

REmember, this page gets created when web inspector opens.  REsponsible for creating the custom tab in web inspector.  If needed, permissions i saw earlier will be displayed inline, instead of osmewhere else.

`devtools.panels.create(browser.i18n.getMessage("extensionName"))` etc

In order to use this icon, I need to ensure it's part of my project.  

devtools *tab* page will be shown to the user.  So we'll create a js and css file.

* declare manifest items
* create extension tabs
* consider permissions


# Evaluating code
often you evaluate code in the inspected page.  There are a number of wait so to evaluate code.
```js
// Evaluating scripts inside the inspected page

let result = await browser.devtools.inspectedWindow.eval("foo.bar()");
```

Remember, user oculd be inspecting multiple pages at the same time.

By default, the expression is evaluated in the context of the main frame.  But you can specify a different frame

```js
// Evaluating scripts inside a frame in the inspected page

let result = await browser.devtools.inspectedWindow.eval("foo.bar()", {
    frameURL: "http://example.com/",
});
```

e.g. when there are many possible subframes in the page.

Match the apeparanceo f the rest of web inspector (light/dark), use system font family, match colors of more important text by using builtin classes.

I'll create the JS I'll provide to devtools inspected window API.  Evaluaet in the inspected page.  

* Evaluate in inspected page
* Process the results
* Display data in my tab

# User experience
* always create a tab (from the background page)
* user can see where the tabs will appear in web inspector, permissions shown inline
* Use `activeTab` permission to keep as targeted as possible
* match the surrounding style

# Wrap up
* download sample project
* Provide feedback
* Join WebExtensions community group
[[What's new in Safari Web Extensions]]

* https://developer.apple.com/forums/tags/wwdc2022-10100
* https://developer.apple.com/forums/create/question?&tag1=207&tag2=491030
* https://developer.apple.com/documentation/safariservices/safari_web_extensions/adding_a_web_development_tool_to_safari_web_inspector
* https://webkit.org/web-inspector/
* https://developer.apple.com/bug-reporting/
* https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API
* https://developer.apple.com/documentation/safariservices/safari_web_extensions

