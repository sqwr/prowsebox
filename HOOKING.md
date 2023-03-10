# Hooking

## Configuration
The [configswshooker.json](src/configswshooker.json) configuration file contains the list of service workers APIs and events to be monitored. It covers most APIs available to service workers. If there is another API whose calls you would like to monitor, you can add it to the `customapiscalls` section of configuration file. 

The content of the configuration file is shown below
```json
{
	"events": ["install", "activate", "sync", "push", "notificationclick", "pushsubscriptionchange", "message"],
	"apiscalls": {
		"fetch": true,
		"importScripts": true,
		"skipWaiting": true,
		"CacheStorage": ["open", "delete", "match", "has", "keys"],
		"Cache": ["addAll", "add", "put", "delete", "match", "matchAll", "keys"],
		"navigationPreload": ["enable", "disable", "setHeaderValue", "getState"],
		"indexedDB": ["open"],
		"IDBObjectStore": ["add", "put", "clear", "delete", "get", "getAll", "getAllKeys"],
		"Client": ["postMessage"],
		"Clients": ["claim", "get", "openWindow", "matchAll"],
		"PushManager": ["getSubscription", "permissionState", "subscribe"],
		"Crypto": ["getRandomValues"],
		"crypto":  ["getRandomValues"],
		"SubtleCrypto": ["decrypt", "deriveBits", "deriveKey", "digest", "encrypt", "exportKey", "generateKey", "importKey", "sign", "unwrapKey", "verify", "wrapKey"]
	},
	"BroadcastChannel": true,
	"customapiscalls": {
		"eval": true,
		"atob": true,
		"CookieStore.prototype": ["get", "set", "getAll", "delete"]
	},
    "dumpswheadersandcontent": false
}
```

The information that are extracted about APIs include their names, and arguments. For specific arguments, i.e. HTTP requests and responses objects, we take an extract step to serialize them before logging, i.e. extract the url, headers, etc. In the particular case of requests and responses, we do not log their bodies. 


### events
This list includes the common service workers events. Among those are the service worker lifecycle events such as [install]() fired during the service worker installation phase, [activate]()but also functional events such as [fetch](https://developer.mozilla.org/en-US/docs/Web/API/FetchEvent), [message](), [push](), [sync](), [periodicsync](), [notificationclick](), [notificationclose](), [pushsubscriptionchange](), etc. 
Events can be registered by calling with [addEventListener](). 
`ProwseBox` logs when the event is registered, as well as when it is fired. Examples logs for the `fetch` event are shown below. 
- Registration
```json
{
    "name": "fetch",
    "description": "registered.self.fetch",
    "argsList": ["..."], // function that handles fetch events
}
```
- `fetch` event occurence (for a request). Example of a navigation request
```json
{
    "name": "fetch",
    "description": "self.fetch",
    "request": {

    }, 
    "response": {

    }
}
```