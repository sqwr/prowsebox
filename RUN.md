# Example run of ProwseBox
1. Get [the code](src/)
2. Install [the necessary tools](INSTALL.md)
3. Obtain a list of sites to analyse. An example is shown in [allsites.csv](allsites.csv). There are many sources where one can find a list of sites: [CrUX](https://developer.chrome.com/docs/crux/bigquery/), [Tranco](https://tranco-list.eu/), [Common Crawl](https://commoncrawl.org/connect/blog/), [Alexa](https://gist.github.com/chilts/7229605), etc.  Be sure to have only a single site or URL per line. If you the list you download includes additional information such as ranking, be sure to remove them and only keep a single URL per line. If the list contains more sites than the set you want to analyze, also be sure to reduce it to the right size. 
4. Edit the [configuration](CONFIG.md) file to suit your particular setup
5. If you want to use a remote command server to distribute the analysis among many machines or for analysis on mobile browsers, download a copy of the code on the command server

## Initialization
These 3 commands must always be run no matter the type of analysis. They create the different directories, and generate the right [SWHOOKING](SWHOOKING.md) file, and the rest of the commands to be launched.  
```bash
mkdir -p  trancoparts swstrancohomes swscontent mitmswstrancohomes profiles documents swcookies webresponses
node gen_swhook.js
python3 gen_swshooker.py
```
Additional commands to be run depends on the environment: server or client. 
- `main_swshookers.sh`: 
- `main_mitmproxies.sh`:
- `main_clients.sh`: 

`parallel -j8 -a main_mitmproxies`
`parallel -j8 -a main_swshookers.sh`
`node server.js 8915`
`parallel -j8 -a main_clients.sh` 


### Mitmproxy [Optional]
The generated `main_mitmproxies.sh` file contains the commands to start multiple mitmproxy servers on different ports. You can run all the proxy servers with `sh main_mitmproxies.sh` or with GNU/Parallel. The commands in this file looks like the following:
```bash
mitmdump --ssl-insecure --set validate_inbound_headers=false -p 8080 -s ./mitmproxies/httptamper.py &>/dev/null & 
```
This commands starts a mitmproxy that listens on the port `8080`


#### Instance
The generated bash `main_swshooker.sh` file contains
```bash
node swshooker.js ./trancoparts/tranco_0_50.csv ./swstrancohomes/swsinfos_0_50.json "127.0.0.1:8080"
```

### Server [Optional]

```bash
node server.js 8915
```

### Client (or Instance)


#### Client
The generated bash `main_clients.sh` file contains a list of commands to start individual clients. The content of this file looks like:
```bash
python3 client.py "127.0.0.1:8080" &
```
The client will fetch a list of sites to analyze, spawn a browser automated by playwright/puppteer or an extension, and redirects its requests to a local mitmproxy listening on port `8080`.  



## Instance

## Client

#### Instance w/ Mitmproxy
In general, we recommand an instance to operate with a client 



## w/ Command Server
The instance becomes a client. 

### Client 
    ```bash
    mkdir -p trancoparts swstrancohomes
    python3 gen_swshooker.py
    node gen_swhook.js
    sh main_swshooker.sh
    ```
## 
```bash

```


### Example
1. Init
```bash
sh MAIN.sh
```
2. Start Instances
```bash
sh main_swshooker.sh
```