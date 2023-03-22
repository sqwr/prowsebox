# Service Worker Hooking
The command ```node gen_swhook.js``` outputs `swhook.js`, which our service worker monitor, which makes extensive usage of the [Proxy API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to passively hook into most service workers events registration, invocation and APIs calls, in order to log their arguments. To generate `swhook.js`, the following files are taken into consideration:
- `swhooking.js` is the boilerplate service worker hooker. It contains placeholders that will be filled depending on the following configuration files.
- `configswshooker.json` contains the list of service workers APIs and events to monitor. By default, most APIs specific to service workers are supported. Under the `customapis` section, additional APIs can be listed. 
```json
{
	"events": ["install", "activate", "fetch", "sync", "push", "notificationclick", "notificationclose", "notificationshow", "pushsubscriptionchange", "message", "periodicsync"],
	"apiscalls": {
		"fetch": true,
		"importScripts": true,
		"skipWaiting": true,
		"CacheStorage": ["open", "delete", "match", "has", "keys"],
		"Cache": ["addAll", "add", "put", "delete", "match", "matchAll", "keys"],
		"navigationPreload": ["enable", "disable", "setHeaderValue", "getState"],
		"indexedDB": ["open"],
		"IDBObjectStore": ["add", "put", "clear", "delete", "get", "getKey", "getAll", "getAllKeys"],
		"Client": ["postMessage"],
		"Clients": ["claim", "get", "openWindow", "matchAll"],
		"PushManager": ["getSubscription", "permissionState", "subscribe"],
		"Crypto": ["getRandomValues", "randomUUID"],
		"crypto":  ["getRandomValues", "randomUUID"],
		"SubtleCrypto": ["decrypt", "deriveBits", "deriveKey", "digest", "encrypt", "exportKey", "generateKey", "importKey", "sign", "unwrapKey", "verify", "wrapKey"]
	},
	"customapiscalls": {
		"FetchEvent.prototype": ["respondWith"],
		"ExtendableEvent.prototype": ["waitUntil"],
		"CookieStore.prototype": ["get", "set", "getAll", "delete"],
		"registration": ["showNotification", "getNotifications", "update", "unregister"],
		"eval": true,
		"atob": true, "btoa": true,
		"encodeURIComponent": true, "decodeURIComponent": true,
		"decodeURI": true, "encodeURI": true,
		"escape": true, "unescape": true,
		"URLSearchParams.prototype": ["get", "entries", "has", "getAll", "keys", "set", "sort", "forEach", "toString", "values", "delete", "append"],
		"EventTarget.prototype": ["removeEventListener"]
	},
	"BroadcastChannel": true,
	"dumpswheadersandcontent": false
}
```
- `config.json`: many parameters in the general configuration file of `ProwseBox` influence the output `swhook.js`. Overall, these parameters specify how `swhook.js` will be injected in service workers context, and how the logs will be collected:
  - **Hooking**: Puppeteer/Playwright automating Chromium browsers have direct access to service workers contexts. They can inject `swhook.js` when new service workers contexts are created, and make it execute first to amend the execution context. For the data collection, they can periodically extract the monitored logs. Note that we observe a couple of limitations of this method . First, when the new context is created, [CacheStorage]() and [CookieStore]() APIs are not yet present in the execution context. To address this limitation, we rely on other APIs that are available right away, and when one of them is invoked, usually at that time the cache and cookies APIs are also available to be proxied. In general, we do not recommend this method, even though it is convenient because it does not require any external tool like an extension or mitmproxy for hooking service workers. For extensions and Mitmproxy, we do not have such a limitation: all APIs can be proxied, and `swhook.js` prepended to the original service worker code: this will ensure that the service worker context is fully amended (APIs and events are proxied) before the execution of the original service worker starts.
  - **Logs collection**: There are 3 main methods for collecting the service workers APIs logs:
    - through Puppeteer/Playwright (Chromium browsers only) directly from the context of service workers. The `swhook.js` exposes the `sendCacheContentToServer` ( and `getDuplicatedCounts`) methods that can be called to extract the data
    - through [broadcast channel](): pages under the scope of the service worker can listen on the chanel name `serviceworkerinformationtobesaved` in order to collect service workers logs. This is our prefered way for collecting the logs: it can be leverages by web pages, but also by content scripts of browser extensions, and even as an alternative to directly accessing service workers contexts when Puppeteer/Playwright automate Chromium browsers.
    - from the [Mitmproxy server](): the logs are POSTed via a fetch call, using the service worker origin suffixed with `/?serviceworkerinformationtobesaved`. These special URLs are detected by the Mitmproxy, and are not forwared to the target web servers: the logs are extracted and saved, and the Mitmproxy directly responds to the request.



## APIs hooking
Most APIs hooking is done usinng the JavaScript Proxy API. Following is how one can passively monitor fetch requests and responses. 
```javascript
fetch = new Proxy(fetch, {
    apply: function(target, thisArg, argsList) {
        // argsList contains the request to fetch
        let response = target.apply(thisArg, argsList);
        // response is the result of the fetch, can be cloned, serialized and logged
        return response;
    }
})
```
The fetch API is a special case, because in general, what is done for most APIis is to simply log the content of `argsList`. Currently, `swhook.js` has a limited number of functions that performs the  hooking of most service worker APIs and events.

## Supported APIs
The APIs that are hooked are summarized as follows.

| API | Monitored arguments or properties | Description |`swhook.js` function |
|--|--|--|--|
[addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) | [install](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/install_event), [activate](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/activate_event), [fetch](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/fetch_event), [message](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/message_event), [push](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/push_event), [sync](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/sync_event), [notificationclick](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/notificationclick_event), [notificationclose](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/notificationclose_event), [periodicsync](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/periodicsync_event), [pushsubscriptionchange](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/pushsubscriptionchange_event) | service workers events registration and dispatch | `proxyAddEventListenersOnSelf` |
[fetch](https://developer.mozilla.org/en-US/docs/Web/API/fetch) | | fetch calls | `proxyAndRegisterArgumentsFetch` |
[Cache](https://developer.mozilla.org/en-US/docs/Web/API/Cache) / [CacheStorage](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage) | [Cache](https://developer.mozilla.org/en-US/docs/Web/API/Cache).[[match](https://developer.mozilla.org/en-US/docs/Web/API/Cache/match),[matchAll](https://developer.mozilla.org/en-US/docs/Web/API/Cache/matchAll),[add](https://developer.mozilla.org/en-US/docs/Web/API/Cache/add),[addAll](https://developer.mozilla.org/en-US/docs/Web/API/Cache/addAll),[put](https://developer.mozilla.org/en-US/docs/Web/API/Cache/put),[delete](https://developer.mozilla.org/en-US/docs/Web/API/Cache/delete),[keys](https://developer.mozilla.org/en-US/docs/Web/API/Cache/keys)], [CacheStorage](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage).[[match](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage/match),[has](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage/has),[open](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage/open),[delete](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage/delete),[keys](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage/keys)] | operations on the cache | `proxyAndHandleArguments` | 
[importScripts](https://developer.mozilla.org/en-US/docs/Web/API/WorkerGlobalScope/importScripts) | | import additional scripts  in service workers contexts | `proxyAndRegisterArguments` |
[skipWaiting](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/skipWaiting) | | make active a waiting service worker | `proxyAndRegisterArguments` |
| [indexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB) | [IndexedDB.open](https://developer.mozilla.org/en-US/docs/Web/API/IDBFactory/open) | connection to an indexeDB database | `proxyAndRegisterArguments` |
| [IDBObjectStore](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore) | [get](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/get), [add](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/add), [put](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/put), [getKey](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/getKey), [getAll](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/getAll), [getAllKeys](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/getAllKeys), [clear](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/clear), [delete](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/delete) | operations on an indexedDB database | `proxyAndRegisterArguments` |
| [PushManager](https://developer.mozilla.org/en-US/docs/Web/API/PushManager) | [getSubscription](https://developer.mozilla.org/en-US/docs/Web/API/PushManager/getSubscription), [permissionState](https://developer.mozilla.org/en-US/docs/Web/API/PushManager/permissionState), [subscribe](https://developer.mozilla.org/en-US/docs/Web/API/PushManager/subscribe) | Manage push notifications subscriptions | `proxyAndRegisterArguments` |
| [Client](https://developer.mozilla.org/en-US/docs/Web/API/Client) | [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Client/postMessage) | send a message to a service worker client (i.e. page, workers) | `proxyAndRegisterArguments` |
| [Clients](https://developer.mozilla.org/en-US/docs/Web/API/Clients) | [get](https://developer.mozilla.org/en-US/docs/Web/API/Clients/get), [matchAll](https://developer.mozilla.org/en-US/docs/Web/API/Clients/matchAll), [claim](https://developer.mozilla.org/en-US/docs/Web/API/Clients/claim), [openWindow](https://developer.mozilla.org/en-US/docs/Web/API/Clients/openWindow) | access service workers clients: pages and workers | `proxyAndRegisterArguments` |
| [ServiceWorkerRegistration](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration) | [showNotification](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/showNotification), [unregister](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/unregister), [update](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/update), [getNotifications](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/getNotifications) | Manage a service worker registration | `proxyAndRegisterArguments` |
| [NavigationPreload](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager) | [enable](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/enable), [disable](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/disable), [setHeaderValue](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/setHeaderValue), [getState](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/getState) | preload navigation requests | `proxyAndRegisterArguments` |
| [CookieStore](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore) | [get](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore/get), [set](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore/set), [delete](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore/delete), [getAll](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore/getAll) | getting/settings cookies | `proxyAndRegisterArguments` |
[removeEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener) | [install](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/install_event), [activate](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/activate_event), [fetch](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/fetch_event), [message](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/message_event), [push](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/push_event), [sync](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/sync_event), [notificationclick](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/notificationclick_event), [notificationclose](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/notificationclose_event), [periodicsync](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/periodicsync_event), [pushsubscriptionchange](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/pushsubscriptionchange_event) | remove a registered handler for an event  | `proxyAndRegisterArguments` | 
| [Crypto](https://developer.mozilla.org/en-US/docs/Web/API/Crypto) | [getRandomValues](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues), [randomUUID](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID) | random values generation | `proxyAndRegisterArguments` |
| [SubtleCrypto](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto) | [encrypt](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/encrypt), [decrypt](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/decrypt), [sign](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/sign), [verify](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/verify), [digest](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest), [generateKey](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/generateKey), [deriveKey](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/deriveKey), [deriveBits](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/deriveBits), [importKey](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/importKey), [exportKey](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/exportKey), [wrapKey](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/wrapKey), [unwrapKey](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/unwrapKey) | cryptographic primitives | `proxyAndRegisterArguments` |
| [eval](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval) | | evaluates JavaScript code represented as string | `proxyAndRegisterArguments` |
| [atob](https://developer.mozilla.org/en-US/docs/Web/API/atob) | | decodes a base64 string | `proxyAndRegisterArguments` |
| [btoa](https://developer.mozilla.org/en-US/docs/Web/API/btoa) | | creates a base64 string | `proxyAndRegisterArguments` |
| [decodeURIComponent](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/decodeURIComponent) | | decodes encoded URLs components | `proxyAndRegisterArguments` |
| [encodeURIComponent](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) | | encode URLs components| `proxyAndRegisterArguments` |
| [decodeURI](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/decodeURI) | | decodes encoded URLs | `proxyAndRegisterArguments` |
| [encodeURI](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/escape) | | encodes URIs | `proxyAndRegisterArguments` |
| [escape](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/escape) | | escape strings | `proxyAndRegisterArguments` |
| [unescape](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/unescape) | | unescape strings | `proxyAndRegisterArguments` |
| [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) | [append](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/append), [delete](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/delete), [entries](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/entries), [forEach](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/forEach), [get](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/get), [getAll](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/getAll), [has](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/has), [keys](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/keys), [set](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/set), [sort](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/sort), [toString](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/toString), [values](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/values) | manipulating URLs arguments | `proxyAndRegisterArguments` |


## Collected Data Structure
The logs collected for each 
```javascript
{
    name: string, // name of the API/event
    description: string, // unique string to distinguish an API/event from others
    argsList: [ arg1, ...] // arguments of APIs calls...
        || { name: string, callback: string }, // event handler function name and callback string
    event: { // relevants to events
        data: any // relevant for message, push, sync, notificationclick events
        origin: string // relevant to message event
        type: string // same as event name
    },
    request: { // relevant to fired fetch events, fetch calls, and cache APIs that handle requests(i.e. caches.match)
        url: string, // request url
        headers: {
            name1: value1,
            name2: value2,
            ...
        },
        destination: string, // request type: document, image, script, etc.
		credentials: string, // same-origin, omit, include
		mode: string, // request mode: navigate, cors, no-cors, same-origin, websocket
		method: string // request method: get, post, put, ...
		referrer: string, // request referrer: about:client, client, a URL, ...
		integrity: string, // request integrity 
		redirect: string, // request redirect
    }, 
    response: { // relevant to fetch calls and cache operations that handle responses (cache.put)
        headers: {
            name1: value1,
            name2: value2,
            ...
        },
        url: string, // response url if available
        ok: boolean, // true for 200+ responses status codes
        type: string, // response type: basic, cors, opaque, etc.
        status: number, // response status code: i.e. 200, etc.
        statusText: string, // response statusText: i.e. OK
        byteLength: number // response size (for same-origin and CORS-compliant responses)
    }, 
    timestamp: number // Value of Date.now() when the API is logged
}
```
where `name` and `description` are help identify 
The chosen `description` for an API or event must be unique because it is the distinguishing key when APIs or events have the same `name`, i.e. `get`, `put`, `add` that appears on many APIs

### Events
Two separate logs are recorded for events, when they are registered, and when they are fired. Event handlers can be setup by calling the `addEventListener` global method, or by the setting the `onEVENTNAME` object (where `EVENTNAME` is the event name, i.e. `install`, `fetch`, or `message`).
1. The structure of the log during an event registration is as follows: 
```javaScript
{
    name, description,
    argsList: { name, body },
    timestamp
}
```
- `name` is of the form `self.EVENTNAME`
- `description` of the form `registered.self.EVENTNAME` 
- `EVENTNAME` is any of the events, i.e. `install`, `fetch`, `activate`, `push`, `message`, etc.
- `argsList.name` is the name of the event handler function (or the empty string if this is an anonymous function) and 
- `argsList.body` is the event handler function body turned into a string.
2. The structure of the event log when it is fired is as follows:
```javaScript
{
    name, description, 
    event: { data?, origin?, type },
    request?: {} // serialized request information for fetch events
}
```
- `name` is of the form `EVENTNAME`
- `description` of the form `self.EVENTNAME` 
- `EVENTNAME` is any of the events, i.e. `install`, `fetch`, `activate`, `push`, `message`, etc.
- `event.type` is basically the same as `EVENTNAME` but is read from the fired event object itself.
- `event.data` relevant for `message` and `push` events. For push events, the `event.data` is the text representation of the data.
- `event.origin` relevant for `message` events
- `request` object is only relevant for `fetch` events 

It is worth mentionning that, as we monitor all events registered through `addEventListener` are also captured even though they are not listed in the Table above. The only limitation is when those events are registered by setting their related `onEVENTNAME` handler function global property.


### Fetch API
The [fetch event](https://developer.mozilla.org/en-US/docs/Web/API/FetchEvent) logged according to the description done in the [events](#events) section above, is to be distinguished from the [fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) effectively used for issuing requests to download resources from a remote server. The logs for the fetch API have the following structure:
```javascript
{
    name: "fetch", "description": "fetch",
    request: { ... }, response: { ... }
}
```
The serialized request and response do not include the bodies (to save storage space)

### Cache operations
Two main APIs perform operations on the cache storage. 
- For [CacheStorage](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage), its methods calls are logged as follows:
  - `name` is the name of the method: `match`, `has`, etc.
  - `description` is the name of the method prefixed with `CacheStorage.prototype`: i.e. `CacheStorage.prototype.match`, `CacheStorage.prototype.has`
- For [Cache](https://developer.mozilla.org/en-US/docs/Web/API/Cache), its methods calls are logged as follows:
  - `name` is the name of the method: i.e. `addAll`, `put`
  - `description` is the name of the method prefixed with `Cache.prototype`: i.e. `Cache.prototype.addAll`, or `Cache.prototype.put`
  
Most of these methods of these APIs handle request objects, and some such as [Cache.put](https://developer.mozilla.org/en-US/docs/Web/API/Cache/put) are additionally passed response objects. In that case, the request/response are serialized and added to the log. For methods that do not handle request or response objects, the `argsList` entry is added to the log with the list of the method invocation arguments. 

### Other APIs
For all other APIs, the `argsList` entry is always added to the log, which holds the arguments passed to the API when it was invoked. Notably, we do not log the arguments of the [SubtleCrypto](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto) methods (because those are usually large bytes of array which occupy a log of storage space).
- For all APIs, the `name` entry is the name of the API, no matter if it is a global API (i.e. `eval`, `atob` or a method under another API)
- For the `description` entry:
  - its value is the name of the API (when it is a global API, i.e. `importScripts`, `eval`, `atob`, etc.). For the `skipWaiting` API, the description is prefixed with `self.`
  - or the name of the method prefixed with its parent object name: i.e. `registration.showNotification`, `indexedDB.open`. In a good number of cases, the name of the parent object is prefixed with `prototype`. That is the case for: `NavigationPreload`, `PushManager`, `IDBObjectStore`, `URLSearchParams`, `Clients`, `CookieStore`, `ExtendableEvent`, `FetchEvent`
  

### Custom APIs
When adding custom APIs, make sure that they exist. In particular, when you are listing methods to be monitored, check if the methods are properties defined directly under the parent object, or if they are part of the parent object prototype. In this case, suffix the parent object with `prototype`. The excerpt of `configswshooker.json` shows various APIs we have as custom apis calls. 
```json
{
    "customapiscalls": {
		"FetchEvent.prototype": ["respondWith"],
		"ExtendableEvent.prototype": ["waitUntil"],
		"CookieStore.prototype": ["get", "set", "getAll", "delete"],
		"registration": ["showNotification", "getNotifications", "update", "unregister"],
		"eval": true,
		"atob": true, 
		"btoa": true,
		"decodeURIComponent": true,
		"decodeURI": true,
		"encodeURI": true,
		"URLSearchParams.prototype": ["get", "entries", "has", "getAll", "keys", "set", "sort", "forEach", "toString", "values", "delete", "append"]
	}
}
```

# Hooking Methods
We leverage 3 main methods for hooking service workers, i.e. injecting and making it execute first, the `swhook.js` file in an original service worker code.

## Mitmproxy
[Mitmproxy](https://mitmproxy.org/) is an efficient, open-source scriptable HTTPS proxy. 
Most modern, desktop and mobile browsers, provide the ability to redirect all HTTPS requests to a proxy, provided that the proxy Certificate Authority (CA) has been added to the list of CAs trusted by the browser or the device. Mitmproxy makes is possible to write addons (Python3 scripts) to manipulate requests before they are forwarded to the origin server, manipulate responses before they are delivered to the requesting client, or respond directly to requests when they are intercepted, without reaching the target origin server. See [src/mitmproxies/httptamper.py] for the addon used to manipulate HTTPS requests/responses at the level of Mitmproxy, for among other things, injecting `swhook.js` in service workers responses. 
We recommend the Mitmproxy method for service workers hooking. It is the most universall solution, as it is supported by most browsers. The entries in the [configuration file](CONFIG.md) that enables the Mitmproxy include:
- `mitmproxy`: its default and currently sole value is `mitmdump`. `mitmproxy` and `mitmweb` are alternate binaries, but we have not tested how the the addon works with them. We have also tested [Mockttp](https://github.com/httptoolkit/mockttp)  as an alternative, but we have not kept in in the current version of the framework. When any other hooking method is used, or when you do not want to use Mitmproxy, make sure to remove the `mitmproxy` entry in the [configuration](CONFIG.md) file
- `mitmaddress`: the IP address of the mitmproxy server. In our experiments, we have deployed local Mitmproxy servers, i.e. `127.0.0.1` is a suitable IP address. Nothing prevents you from deploying a remote Mitmproxy server.
- `mitmiport`: the (initial) port of the Mitmproxy proxy server. By default, this is `8080`
- `mitmproxies`: the number of concurrent Mitmproxy proxy servers you want to deploy, i.e `8`. The framework will generate the `main_mitmproxies.sh` file that contains `8` Mitmproxy servers, the first one on port `8080`, the second on `8081` and the 8th one on port `8087` (provided that the initial port number is `8080` as in our example)
- `swshooking`: set this boolean to `true`



### Fetch metadata
It is important to inject `swhook.js` only in servcie worker responses, which we recall, as JavaScript programs. To precisely isolate service workers responses, we extensively rely on the [Sec-Fetch-Dest](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Dest) request header of [Fetch metadata](https://developer.mozilla.org/en-US/docs/Glossary/Fetch_metadata_request_header), which is largely supported by modern browsers. Safari browsers are a notable exception

### No fetch metadata
For simplicity, we could have hooked all JavaScript programs ,by inspecting the  [content-type response header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types#textjavascript) for JavaScript MIME types. As one may guess, this solution adds a lot of overhead and potential breakage to websites (i.e. integrity checks on JavaScript responses). In the end, the solution we divised to hook service workers on these browsers that do not support fetch metadata works as follows:
- We inject a code in HTML pages, which hooks for service workers registration, and directly report the service worker to Mitmproxy. 
- As all requests and responses go through the Mitmproxy, when a response for that service worker URL is received, Mitmproxy hooks it.
This solution has to be explicitely configured by settings the entries of the `nofetchmetadata`,  `swsurltomitm` and `mitmhtmhooking` to `true`. 


## Puppeteer/Playwright
Service workers are supported by Puppeteer and Playwright for Chromium browsers. We leverage this feature for hooking service workers. For Playwright (the process is similar for Puppeteer), we listen for [serviceworker events](https://playwright.dev/docs/api/class-browsercontext#browser-context-event-service-worker), get a handle to the service worker target and [evaluateHandle](https://playwright.dev/docs/api/class-worker#worker-evaluate-handle) method to inject our hooks in the service worker context. 

### Limitations
We have observed that the [cache](https://developer.mozilla.org/en-US/docs/Web/API/Cache) and the [cookiestore](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore) APIs are usually not yet present in the service worker context when we use this method of hooking. To workaround this limitation, we proxy them as soon as any of the other APIs (which are present at the time of hooking) is triggered. Nonetheless, even though it is unlikely that cache operations occur APIs such as importScripts or events such as install, activate or fetch are fired, this is a limitation to bear in mind when using this method. Another question we have not addressed yet for this method of hooking, concerns the update of service workers: we do not know if updated service workers will be successfully monitored. 

## WebRequest API
This method is only supported by Firefox browsers, on Desktop and Android. Firefox webExtensions can use the [webRequest](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/webRequest/filterResponseData) to modify HTTP responses. We leverage this with the additional  `webRequestFilterResponse.serviceWorkerScript` permission to hook service worker responses by prepending the content of `swhook.js`. As Firefox also support [fetch metadata](#fetch-metadata), we aditionally filter service worker responses based on these headers. Note that, as they are not related to any tab or window, service workers requests get assign `-1` as their `tabId`. We leverage all these information to precisely isolate service worker responses and only hook those. 


# Service workers logs collection
Collecting the service workers logs present a couple of interesting challenges. The solution we retained are described below, and hugely depends on the way the service workers are hooked

## BroadcastChannel
[BroadcastChannel](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API) makes it possible to communicate between same-origin browsing contexts. Basically, web pages, extensions content scripts can listen on a named channel, i.e. `serviceworkerinformationtobesaved`, and whenever a service worker has information to save, it just writes them on the channel of that name. We leverage this API to post the service workers APIs logs collected by `swhook.js`. And from navigated webpages, or extensions content scripts, the data is received and relayed to the automation party (i.e. background script for an extension or Puppeteer/Playwright in the case of CDP). 
 
## CDP

