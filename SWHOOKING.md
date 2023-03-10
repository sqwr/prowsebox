# Service Worker Hooking
The command ```node gen_swhook.js``` outputs `swhook.js`. 

## `configswhooking.json`

### Events
| Event name | Description | Reference | Multiple registrations
|--|--|--|--|
`install` | 
`activate` | 
`fetch` | 
`message` | 

An event can either be registered by invoking the `addEventListener` global method, passing the name of the event and a handler (callback) function as arguments. 
```javascript
addEventListener('install', (event) => {
    // Precache content here
});
```

Otherwise, one can also set the event handler global property. 
```javascript
oninstall = (event) => {
    // Precache content here
}
```
`Prowsebox` records when an event is registered and each time the event is effectively fired

#### 



### APIs calls
### Custom API calls


## `swhooking.js`
`proxyAndRegisterArguments`

## `config.json`

| parameter |||
|--|--|--|
`mitmproxy` | 
`postcollectedata` | 
`browser` | 
`automation` | 
`firenotificationclick` | 
`firenotificationclose` | 
`setup`


## `configswhook.js`



||||
|--|--|--|
`importScripts` | 
`addEventListener` |
`CacheStorage.prototype` | 
