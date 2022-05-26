You get the *benefit* of being on the appstore lmao

# Working with Xcode
```bash
xcode-select --install
```

```bash
xcrun safari-web-extension-converter path
xcrun safari-web-extension-converter --rebuild-project foo
```

> Warning: persistent background pages are not supported on iOS and iPadOS.

Manifest.  background: `"persistent": false`

Enable extension in settings.  Then it appears in the Aa menu in Safari.


# Permissions and privacy
* Users are in control
* Permissions are per-website
* Four types of extension permissions
	* Script injection
	* Implicit permission.  Sensitive vs nonsensnitive.  
	* Explicit permissions.  Prompt always shown, not badged.  
	* Active tab.  

```js
let titleContent = document.querySelector ('meta[property="og: title"]' )?.content;
let descriptionContent
= document. querySelector ('meta[property="og: description"]' )?.content;
3
let imageURL = document.querySelector ('meta[property="og: image"])?.content:
5
browser.runtime.onMessage.addListener((request,
sender, sendResponse) =>
console.log("Received request:
request);
3):
```

```html
<!DOCTYPE html>
chtml>
chead>
<meta charset="UTF-g">
<link rel="stylesheet" href="popup.css">
‹script type="module" src="popup.js"></script>
</head>
<body>
<h1 id="title">Title</h1»
<p id="description">Description</p›
Kim id="image">
</body>
</html>
```

```js
let titleElement = document.getElementById('title');
let descriptionElement = document.getElementById('description') ;
let imageElement = document.getElementById('image') ;
let tabs = await browser.tabs. query((active: true, currentWindow: true});
let response = await browser.tabs.sendMessage(tabs[0].id, {update: "please"}) ;
titleElement.textContent = response?. title;
descriptionElement.textContent=response?.description;
imageElement.src = response?.image;

```

Safari preferences, open develop menu.  Then in develop menu can open web inspector from the simulator.


# Submitting to App Store
## Tips
* Code and content must be your own
* Use a custom icon and unique name
* Provide significant value to the user
* No web payments or donations

# Wrap up
submit safari web extensions for mac and ios
etc

