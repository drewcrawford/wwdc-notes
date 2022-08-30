#safari 

Remotely send notifications to your web app's users.  

# Questions
* suported in mac safari beginning in macos ventura
* iOS, iPadOS, **next year**.
* old APIs still work

**It's really Web Push**.

**NO site changes required**

Now would be a great time to switch from browser detection, to feature detection.

**NO apple developer account is required**.

Make sure to allow `*.push.apple.com`.  If you tightly manage endpoints.

# User experience
Need to keep up to date with the webkit OSS project.  WK.org is not allowed to request permission to push, without the user asking with a gesture.

System notification prompt, same as non-browser apps.

Get notified as new blockposts and new commits.  Being notified will distract me from important work.  Check that box.

Once a user has granted permissions, they maintain control.  I'm used to managing notification preferences inside system settings.  Go in there and see webkit.org's settings.  SAme ocnfiguration as other apps or services.

Can also go to safari settings.

* **Permission requires a user gesture**.
* Users control notifications
	* with websites in safari preferences
	* with apps in system settings
* You provide fine-grained controls

# Web Push flow
User visits website in browser.
It can installa  service worker.  Unit of JS that operates on behalf of an entire domain, separate from the tab.
Once the worker script is isntalled, your app is eligible to resppond to a push subscription.

Tied to a user gesture.  When your site asks for a subscription, user sees this system prompt.  

User might deny the request.  JS should be prepared to handle that.  Assuming the user grants permission, you get back a push subscription object.

You send this payload back to the server in whatever manner works best.  many server packages have web push support to manage subscriptions, or roll your own.  How/when to actually send a push message.

Once your server has sent a push message, safari wakes up your service worker and sends it a JS push event.  Showing notification to the user is a requirement while handling the push event.  

Happens even if safari is not running.  Click event sent to service worker, e.g. opening a new window.
# Implementation details


```js
// BrowserPetsWorker.js

function handleMessageEvent(event) {
    // ...
};
self.addEventListener('message', (event) => {
    handleMessageEvent(event);
});

function primeCaches() {
    // ...
};
self.addEventListener('install', (event) => {
    primeCaches();
});

self.addEventListener('fetch', (event) => {
    event.respondWith(caches.match(event.request));
});
```

Wehn you visit in a tab, this excerpt registers the service worker if necessary.  Feature detection.

```js
// BrowserPetsMain.js

var registration;
if ('serviceWorker' in navigator) {
    let registration = await navigator.serviceWorker.getRegistration();
    if (!registration)
        registration = await navigator.serviceWorker.register('BrowserPetsWorker.js');
}
```

You cannot request push permission without a user gesture.  onClick is one of many ways.


```js
// BrowserPetsMain.js

async function subscribeToPush() {
    // ...
}

// BrowserPetsMain.html

<button onclick="subscribeToPush()">Register for Updates</button>
```

Our server uses a public key.  Here, we use the standard technology called `VAPID`.  I won't go over these details, but there are resources to help you with the best solution for your server's setup.

We are promising to always make pushes user-visible.  While standard optionally accommodates silent JS runtime, most browsers do not support that.  Safari does not support that.  Browserpets does not need that.

```js
// BrowserPetsMain.js

async function subscribeToPush() {
    let serverPublicKey = VAPID_PUBLIC_KEY; 

    let subscriptionOptions = {
        userVisibleOnly: true,
        applicationServerKey: serverPublicKey
    };

    let subscription = await swRegistration.pushManager.subscribe(subscriptionOptions);

    sendSubcriptionToServer(subscription);
}
```

```js
// BrowserPetsMain.js

async function subscribeToPush() {
    let serverPublicKey = VAPID_PUBLIC_KEY; 

    let subscriptionOptions = {
        userVisibleOnly: true,
        applicationServerKey: serverPublicKey
    };

    let subscription = await swRegistration.pushManager.subscribe(subscriptionOptions);

    sendSubcriptionToServer(subscription);
}
```

Wordpress already has support for standard web push.  Resources to find the right solution for any setup.



```js
// BrowserPetsMain.js

async function subscribeToPush() {
    let serverPublicKey = VAPID_PUBLIC_KEY; 

    let subscriptionOptions = {
        userVisibleOnly: true,
        applicationServerKey: serverPublicKey
    };

    let subscription = await swRegistration.pushManager.subscribe(subscriptionOptions);

    sendSubcriptionToServer(subscription);
}
```

We use JSON accessor here.  When we subscribed for push, we promised to make it user visible.  So we always show a notification in response for each push.  Do this as early as possible in your event handler.


```js
// BrowserPetsWorker.js

self.addEventListener('push', (event) => {
    let pushMessageJSON = event.data.json();

    // Our server puts everything needed to show the notification
    // in our JSON data.
    event.waitUntil(self.registration.showNotification(pushMessageJSON.title, {
        body: pushMessageJSON.body,
        tag: pushMessageJSON.tag,
        actions: [{
            action: pushMessageJSON.actionURL,
            title: pushMessageJSON.actionTitle,
        }]
    }));
}
```
Need to handle the user clicking on it.  In this notification click handler, browserepts will take the URL from the notification to open a new window.

```js
// BrowserPetsWorker.js

self.addEventListener('notificationclick', async function(event) {
    if (!event.action)
        return;

    // This always opens a new browser tab,
    // even if the URL happens to already be open in a tab.
    clients.openWindow(event.action);
});
```

that's it.  

# Development and debug best practices
* web inspector
	* service workers
	* break on event handlers
* server errors

# Privacy and power
* **permission requires a user gesture**
* Require the user actually ask to eanble web push.
* No silent background runtime
	* Violations result in revoked subscriptions
	* in beta, 3 push events.

# Wrap up
* It's really web push
* any site can use it
* code to the standards

[[What's new in Safari and WebKit]]


* https://developer.apple.com/forums/tags/wwdc2022-10098
* https://developer.apple.com/forums/create/question?&tag1=205&tag2=420030
* https://developer.mozilla.org/en-US/docs/Web/API/Push_API
* https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API
* https://developer.apple.com/documentation/usernotifications/sending_web_push_notifications_in_safari_and_other_browsers
* https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
* https://developer.apple.com/bug-reporting/
