# Structure of collected data
`ProwseBox` generates data in the following folders (See the [config.json](CONFIG.md) file for more details)

## config.io.outdir
This is the main folder holding the most important data about service workers. By default, this folder is named `swstrancohomes/` (because we started this tool with [Tranco list](https://tranco-list.eu/)). It is where each `ProwseBox` instance stores data about links it has navigated, the service workers urls, their APIs calls, DOMs information, content stored in the cache, etc. Each file in this folder is a JSON.stringified array (currently consisting of 17 items) described as follows (we start at position `0`). When offline mode is enabled, there is for each file in the folder, a second `<FILE>.offline` which contains the data for the offline analysis. Both have the same structure. 
### 0
Navigated urls object `{}` associating a URL to navigate, i.e. `https://google.com` to the resulting navigated URL, i.e. `https://www.google.com/`. When there is an error during the navigation (i.e. timeout, network error), the resulting navigated URL is set to the string `error`. Example of the object is 
```json
{ 
    "https://google.com": "https://www.google.com/", 
    "https://twitter.com": "https://twitter.com/",
    "https://youtube.com": "https://www.youtube.com/",
    "https://example.invaliddomain.com": "error" 
}
```
### 1
Request information object `{}` grouped by initiator, type, url, header names and values. Note that in order to save space, we remove the `user-agent` header from the request information that we save
```json
{
    "about:blank": { // initiator, i.e. about:blank, https://www.example.com/, 
        "document": { // request type, i.e document, script, image, stylesheet, etc.
            "https://www.example.com/": { // request url
                "referer-policy": "origin-when-cross-origin" // header name associated to value
            }
        }
    },
    ...
}
```

### 2 
Response information object `{}` grouped by initiator, request type, response url, header names and values. 
```json
{
    "about:blank": { // initiator, i.e. about:blank, https://www.example.com/, 
        "document": { // request type, i.e document, script, image, stylesheet, etc.
            "https://www.example.com/": { // response url
                "content-security-policy": "script-src 'self'; object-src 'none'; default-src 'none'; frame-ancestors 'self'" // header name associated to value
            }
        }
    },
    ...
}
```

### 3
Registered service workers urls and execution errors. There are also entries that are simply there for legacy and compatibility purposes (`ProwseBox` has been used in other projects)
```json
{
    "urls": {  // registered service workers urls
        "https://www.example.com/sw.js": "",
        ...
    }, 
    "errors": { // errors reported by Chrome DevTools Protocol during service workers execution. We report here common errors
        "Failed to register a ServiceWorker for scope (...) with script (...)": "", 
        "Uncaught NetworkError: Failed to execute 'importScripts' on 'WorkerGlobalScope': The script at ... failed to load.": "", 
    },
    "swr": {}, "prb": {}, "coverage": {}, "updates": {}, "versions": {} // legacy and compatibility entries
}
```

### 4
An empty string: this entry is simply there for legacy and compatibility reasons. It 
```json
""
```

### 5
Monitored service workers APIs and events calls information object `{}`.
```json
{
    "https://www.example.com/sw.js": { // service worker URL
        "25000": { // size of partial data
            "loc": {
                "origin": "https://www.example.com",
                "href": "https://www.example.com/sw.js",
                "scope": "https://www.example.com/",
                "scriptURL": "https://www.example.com/sw.js"
            },
            "clients": { // service workers clients
                "https://www.example.com/": ""
            },
            "data": [ // 
                {...}, {...}
            ]
        }
    }
}
```





## config.io.swscontent

## config.io.documents

## config.io.importedscripts
