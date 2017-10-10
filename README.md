# PWA Training Notes

## Intro to Service Workers
* A proxy for network requests
* Events
... Install, is fired when the SW is new or has been modified. This is a good place for caching static assets.
... Activate, fired after the Install event It's often used to update caches
... Message
* Functional Events: Fetch, Sync and Push
* After initial installation and activation, re-registering an existing worker does not re-install or re-activate the service worker. Service workers also persist across browsing sessions.
* A modified worker isn't activated until the existing service worker is no longer in use. By closing all pages under the old service worker's control, we are able to activate the new service worker.
* The `skipWaiting()` method allows a service worker to activate as soon as it finishes installation. The install event listener is a common place to put the `skipWaiting()` call, but it can be called anywhere during or before the waiting phase. 
* The default scope is the path to the service worker file, and extends to all lower directories. So a service worker in the root directory of an app controls requests from all files in the app.