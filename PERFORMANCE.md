# Estimating ProwseBox performance
The purpose of this document is to help estimate how `ProwseBox` will perform on a specific dataset and [configuration](CONFIGURATION.md). As one may guess, major factors that influence the performance include:
- the number of sites under analysis
- the number of concurrent instances spawned
- the Internet speed
- the average number of sites that deploy service workers, etc. 
The phases that are enabled (`initial`, `discovery`, `coverage`, `offline`) also adds to the overall performance

## Configuration File
```json
{
    "urls": 1000000,
    
}
```
## Phase performance
$ Ceil() $