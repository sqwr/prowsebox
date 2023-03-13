# Service workers events and APIs
Most service workers APIs and events are proxied and logged. 

## General structure

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
where
- `name` and `description` are help identify 
The chosen `description` for an API or event must be unique because it is the distinguishing key when APIs or events have the same `name`.


## Events

| event | registration `name` | registration `description` | `name` when fired | `description` when fired |
|--|--|--|--|--|
[install](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/install_event) | `self.install` | `registered.self.install` | `install` | `self.install` | 
[activate](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/activate_event) | `self.activate` | `registered.self.activate` | `activate` | `self.activate` | 
[fetch](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/fetch_event) | `self.fetch` | `registered.self.fetch` | `fetch` | `self.fetch` | 
[message](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/message_event) | `self.message` | `registered.self.message` | `message` | `self.message` |
[push](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/push_event) | `self.push` | `registered.self.push` | `push` | `self.push` |
[notificationclick](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/notificationclick_event) | `self.notificationclick` | `registered.self.notificationclick` | `notificationclick` | `self.notificationclick` |
[notificationclose](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/notificationclose_event) | `self.notificationclose` | `registered.self.notificationclose` | `notificationclose` | `self.notificationclose` |
[pushsubscriptionchange](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/pushsubscriptionchange_event) | `self.pushsubscriptionchange` | `registered.self.pushsubscriptionchange` | `pushsubscriptionchange` | `self.pushsubscriptionchange` |
[sync](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/sync_event) | `self.sync` | `registered.self.sync` | `sync` | `self.sync` |
[periodicsync](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/periodicsync_event) | `self.periodicsync` | `registered.self.periodicsync` | `periodicsync` | `self.periodicsync` |


- event registration logs structure
```javaScript
{
    name, description,
    argsList: { name, body }
}
```
The `name` is of the form `self.EVENTNAME` and the `description` of the form `registered.self.EVENTNAME` where `EVENTNAME` is any of the events registered through 
- event dispatch (fired) logs structure
```javaScript
{
    name, description, 
    event: { data?, origin?, type },
    request?: {} // serialized request information for fetch events
}
```


