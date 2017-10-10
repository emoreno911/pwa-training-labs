# PWA Training Notes

[Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```

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
* To make a _POST_ request with fetch, we use the `init` parameter to specify the method (similar to how we set the HEAD method in section 5). This is also where we set the `body` of the request. The body is the data we want to send.
* The `FormData` constructor can take in an HTML `form`, and create a `FormData` object. This object is populated with the form's keys and values.
* Using `mode: no-cors` allows fetching an opaque response. This prevents accessing the response with JavaScript (which is why we comment out validateResponse and readResponseAsText), but the response can still be consumed by other API's or cached by a service worker.
* An _opaque_ filtered response is a filtered response whose type is "opaque", url list is the empty list, status is 0, status message is the empty byte sequence, header list is empty, body is null, and trailer is empty.
* Like cross-origin requests, _custom headers_ must be supported by the server from which the resource is requested. Anytime a custom header is set, the browser performs a preflight check. This means that the browser first sends an OPTIONS request to the server, to determine what HTTP methods and headers are allowed by the server. If the server is configured to accept the method and headers of the original request, then it is sent, otherwise an error is thrown.