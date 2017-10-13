# PWA Training Notes

[Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

## Intro to Service Workers
* A proxy for network requests
* Events

  _Install_, is fired when the SW is new or has been modified. This is a good place for caching static assets.

  _Activate_, fired after the Install event It's often used to update caches.

  _Message_.

* Functional Events: Fetch, Sync and Push
* After initial installation and activation, re-registering an existing worker does not re-install or re-activate the service worker. Service workers also persist across browsing sessions.
* A modified worker isn't activated until the existing service worker is no longer in use. By closing all pages under the old service worker's control, we are able to activate the new service worker.
* The `skipWaiting()` method allows a service worker to activate as soon as it finishes installation. The install event listener is a common place to put the `skipWaiting()` call, but it can be called anywhere during or before the waiting phase. 
* The default scope is the path to the service worker file, and extends to all lower directories. So a service worker in the root directory of an app controls requests from all files in the app.


## Working with Fetch API
* The Fetch API is a simple interface for fetching resources.
* Response Read methods: `arrayBuffer()`, `blob()`, `formData()`, `json()` and `text()`.
* _HEAD_ requests are just like GET requests except the body of the response is empty. You can use this kind of request when all you want the file's metadata, and you want or need the file's data to be transported.
* To make a _POST_ request with fetch, we use the `init` parameter to specify the method. This is also where we set the `body` of the request. The body is the data we want to send.
* The `FormData` constructor can take in an HTML `form`, and create a `FormData` object. This object is populated with the form's keys and values.
* Using `mode: no-cors` allows fetching an opaque response. This prevents accessing the response with JavaScript (which is why we comment out validateResponse and readResponseAsText), but the response can still be consumed by other API's or cached by a service worker.
* An _opaque_ filtered response is a filtered response whose type is "opaque", url list is the empty list, status is 0, status message is the empty byte sequence, header list is empty, body is null, and trailer is empty.
* Like cross-origin requests, _custom headers_ must be supported by the server from which the resource is requested. Anytime a custom header is set, the browser performs a preflight check. This means that the browser first sends an OPTIONS request to the server, to determine what HTTP methods and headers are allowed by the server. If the server is configured to accept the method and headers of the original request, then it is sent, otherwise an error is thrown.


## Caching Files with Service Worker
* To serve content from the cache and make your app available offline you need to intercept network requests and respond with files stored in the cache. There are several approaches to this:

  _Cache only_, This approach is good for any static assets that are part of your app's main code (part of that "version" of your app). You should have cached these in the install event, so you can depend on them being there.

  _Network only_, This is the correct approach for things that can't be performed offline, such as analytics pings and non-GET requests.

  _Cache falling back to Network_, If you're making your app offline-first, this is how you'll handle the majority of requests. This gives you the "Cache only" behavior for things in the cache and the "Network only" behaviour for anything not cached (which includes all non-GET requests, as they cannot be cached).

  ```javascript
  self.addEventListener('fetch', function(event) {
    event.respondWith(
      caches.match(event.request).then(function(response) {
        return response || fetch(event.request);
      })
    );
  });
  ```
  _Network falling back to Cache_, This is a good approach for resources that update frequently, and are not part of the "version" of the site (ie.: articles, game leader boards, etc). Handling network requests this way means the online users get the most up-to-date content, and offline users get an older cached version. However, this method has flaws. If the user has an intermittent or slow connection they'll have to wait for the network to fail before they get content from the cache.

  ```javascript
  self.addEventListener('fetch', function(event) {
    event.respondWith(
      fetch(event.request).catch(function() {
        return caches.match(event.request);
      })
    );
  });
  ```
  _Cache then Network_, This is also a good approach for resources that update frequently. This approach will get content on screen as fast as possible, but still display up-to-date content once it arrives.

* Removing outdated caches, Once a new service worker has installed and a previous version isn't being used, the new one activates, and you get an activate event. Because the old version is out of the way, it's a good time to delete unused caches.

  ```javascript
  self.addEventListener('activate', function(event) {
    event.waitUntil(
      caches.keys().then(function(cacheNames) {
        return Promise.all(
          cacheNames.filter(function(cacheName) {
            // Return true if you want to remove this cache,
            // but remember that caches are shared across
            // the whole origin
          }).map(function(cacheName) {
            return caches.delete(cacheName);
          })
        );
      })
    );
  });
  ```
* Network response errors do not throw an error in the fetch promise. Instead, fetch returns the response object containing the error code of the network error. This means we handle network errors in a .then instead of a .catch. However, if the fetch cannot reach the network (user is offline) an error is thrown in the promise and the .catch executes.


## Lighthouse PWA Analysis Tool
* Lighthouse is an open-source tool from Google that audits a web app for PWA features. It provides a set of metrics to help guide you in building a PWA with a full application-like experience for your users.
* Lighthouse tests if your app:
  * Can load in offline or flaky network conditions
  * Is relatively fast
  * Is served from a secure origin
  * Uses certain accessibility best practices


## Working with Promises
* Promises provide a standardized way to manage asynchronous operations and handle errors. Think of a promise as an object that waits for an asynchronous action to finish, then calls a second function. You can schedule that second function by calling `.then()` and passing in the function. When the asynchronous function finishes, it gives its result to the promise and the promise gives that to the next function.

```javascript
var promise = new Promise(function(resolve, reject) {
  // do a thing, possibly async, then...
  if (/* everything turned out fine */) {
    resolve("Stuff worked!");
  }
  else {
    reject(Error("It broke"));
  }
});
promise
  .then(doSomething)
  .then(doSomethingElse)
  .catch(handleError)
```

* A promise is in one of these states:

  _Pending_, The promise's outcome hasn't yet been determined, because the asynchronous operation that will produce its result hasn't completed yet.
  _Fulfilled_, The operation resolved and the promise has a value.
  _Rejected_, The operation failed and the promise will never be fulfilled. A failed promise has a reason indicating why it failed.

* Promise rejections skip forward to the next then with a rejection callback (or catch, since they're equivalent). With `then(func1, func2)`, `func1` or `func2` will be called, never both. But with `then(func1).catch(func2)`, both will be called if `func1` rejects, as they're separate steps in the chain.
* Promise.all, Promise.race.