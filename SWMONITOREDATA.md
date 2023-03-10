# Monitore
```javascript
[ pagesVisited, requestInformations, responseInformations, swsRegs, '', swsData, swsScopes, swsClients, cacheStorage, cookiesStore, swsDuplicates, xData, pagesLinks, pagesVisitedOffline, domInfos, domInfosOffline, pagesLinksOffline, coveragePages ]
```
## pagesVisited
Object literal associating a URL to be navigated to its final URL navigated. The special `error` value indicates an error during URL navigation (i.e. timeout, certificate issue, etc.)
```json
{
    "http://twitter.com": "https://twitter.com/",
    "http://google.com": "https://www.google.com/",
    "http://example.com": "error"
}
```

## requestInformations
An object literal associating requests URLs to their headers. The entries in this object follow the schema 
```json
{
    "top_frame1": {
        "requestType1": {
            "requestURL1": {
                "headername1": "headervalue1",
                "headername2": "..."
            },
            "requestURL2": { ... }
        }, 
        "requestType2": { ... }
    },
    "top_frame2": { ... }
}
```
where:
- `top_frame`: is the parent frame initiating a request. For top navigations tabs, this value is `about:blank`, or `null`, etc. For iframes and subresources (i.e. scripts, images), the `top_frame` is the URL of the page initiating embedding the iframe or loading the subresource
- `requestType`: is the type of the request, i.e. `document`, `image`, `script`. Slight differences may be observed depending on the automation tool used: [Puppeteer/Playwright](https://playwright.dev/docs/api/class-request#request-resource-type), or a [browser extension](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/webRequest/ResourceType).
- `requestURL`, `headername` and `headervalue` are self-explanatory
  
Dumping the headers of all requests can take a lot of space. To avoid that, by default we only log requests of type `document`. The[requestypes](CONFIG.md#requestypes) configuration option can be adjusted to the types of requests to be logged.

## responseInformations
This object contains response headers. The structure is similar to [requestsInformation](#requestinformations). 

## swsRegs
An object holding information about service workers registrations. It structure is as follows:
```javaScript
{
    "urls": {
        "serviceworkerURL1": "",
        "serviceworkerURL2": "",
        "..."
    },
    "aurls": {
        "serviceworkerURL1": []
    }
}
```

# General schema
| entry | Type | Description | Related events/APIs | Examples | 
|--|--|--|--|--|
`name` | string | the name of the API | all | `fetch`, `importScripts` | 
`description` | string | description of the API. Important to distinguish between many APIs with the name name | all | `Cache.get`, etc.
`argsList` | list of API invocation arguments | all | 
`request` | object 
`response` | object 


## Install event
{

}

```json
{

}
{
    "name": "", 
    "description": string,
    "argsList": []
}
```


## Events registration

| en | description | |||||
|--|--|--|--|--|--|--|
`registered.self.install` | 