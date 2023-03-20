# Configuration of ProwseBox
The `config.json` file defines the setup for the analysis of service workers with `ProwseBox`. With the right configuration and the [necessary tools installed](INSTALL.md), collecting service workers data can be fully automated. We start by listing all the possible entries in the configuration file, most of them are optional or applies only to a specific setup. Then, we give a detail explanation about them, and finally, we give example configurations covering most of the setups that `ProwseBox` provides. 

The content of the configuration file is shown below

```json
{
 	"grouping": 100,
 	"discoverypages": 10,
 	"coveragepages": 10,
 	"paralleljobs": 16,
 	"parallelbin": "parallel",
	"mitmproxy": "mitmdump",
	"headless": true,
	"mitmaddress": "127.0.0.1",
	"mitmiport": 8080,
	"mitmproxies": 16,
	"swscontent": true,
	"swshooking": true,
	"postcollectedata": false,
 	"simultaneoustabstransition": 15000,
 	"simultaneoustabs": 10,
	"offlinesimultaneoustabstransition": 10000,
 	"offlinesimultaneoustabs": 30,
	"timeoutabopen": 60000,
	"timeoutabopencoverage": 30000,
	"timeoutnavigation": 60000,
	"blockcontent": true,
	"coverage": true,
 	"discovery": true, 
 	"offline": true,
 	"browser": "chromium",
 	"automation": "playwright",
 	"customprofile": true,
 	"compatibility": true,
	"requestypes": { "document": true },
	"customsiteswithsws": "../allswsorigins.json",
 	"useragent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.5414.119 Safari/537.36",
 	"custombrowserbinary": "/usr/bin/google-chrome-stable",
    "io": {
 		"sites": "../allsites.csv",
 		"indir": "./trancoparts/",
 		"outdir": "./swstrancohomes/",
 		"profilesdir": "./profiles/",
 		"swscontent": "./swscontent/",
 		"documents": "./documents/",
 		"swcookies": "./swcookies/",
 		"indirprefix": "tranco_",
 		"outdirprefix": "swsinfos_",
 		"indirsuffix": ".csv",
 		"outdirsuffix": ".json",
 		"profileprefix": "profile_",
 		"server": {
			"parts": "main_serverparts.sh",
			"trancoparts": "https://commands.doitsec.net/trancoparts",
		   "swstrancohomes": "https://commands.doitsec.net/swstrancohomes",
		   "customsiteswithsws": "https://commands.doitsec.net/customsiteswithsws",
			"localhost": "localhost",
			"localport": 8915,
			"client": false
		},
	   "mobile": {
		   "clientserverhome": "https://commands.doitsec.net/home",
		   "clientserverdatasuffix": "mobileserviceworkerinformationtosave"
	   },
		"mitmdomainstoexclude": {
			"commands.doitsec.net": "commands.doitsec.net"
		}
 	},
 	"setup": "prowsebox" // or sqwr
}
```

## swscontent
A boolean value (to set to `true`) in order to enable the logging of service workers code.
See [swscontent](SWMONITOREDATA.md#swscontent) for more details about the structure of logged service workers code.

## importedscripts
A boolean value (to set to `true`) in order to enable the loggin of service workers code
See [importedscripts](SWMONITOREDATA.md#importedscripts) for more details about the structure of logged imported scripts code.

## webmanifests
A boolean value (to set to `true`) in order to enable the logging of web manifests content.
See [webmanifests](SWMONITOREDATA.md#webmanifests) for more details about the structure of logged web manifests content.

## io
Under this 


## customprofile
An optional boolean indicating whether a custom browser profile should be created to store the (temporary) user data (cookies, service workers codes, cached resources, etc.). With Playwright, this is equivalent to  [launching a persistent content](https://playwright.dev/docs/api/class-browsertype#browser-type-launch-persistent-context) when creating the browser instance to handle the automation.  
- `false`: default value
- `true`: a custom folder, usually created under `profiles` (or any other folder if you have configured it differenty)



## grouping

## headless
Boolean value to launch the browser in headless (default) or headful mode. This works when [automation](#automation) is done with `playwright` or `puppeteer`. Indeed, other automation mechanisms 
  - `true`: (default) the browser runs in headless mode
  - `false`: the browser is spawn in headfull mode


## automation
The tool to use for automation. Its values can be 
- `playwright`: [Playwright](https://playwright.dev). Works for desktop Firefox, Safari and chromium browsers (Chrome, Edge, Brave, Avast, except Opera), Chrome on Android
- `puppeteer`: [Puppeteer](https://pptr.dev/). Works only for desktop chromium browsers (except Opera)
- `extension`: use a [WebExtension](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/webRequest) to automate the analysis. 
  - Chromium browsers. This has to 
  - Firefox: the [web-ext](https://extensionworkshop.com/documentation/develop/getting-started-with-web-ext/) NPM package is used to automate Firefox on Desktop and Android. The Firefox on Android requires [ADB](https://developer.android.com/studio/command-line/adb). 
  
## browser
The type of browser. This key is optional 
- `chromium`: (default) for Chromium browsers
- `firefox`: for firefox browsers
- `safari` or `webkit` for Safari browsers

## platform
The type of platform
- `desktop`: (default)
- `android`: android platform
  - for Chrome `browser`
  - Firefox browser on Android with [web-ext]()


## mitmproxy
If and which HTTPs proxy to use to intercept HTTP requests/responses, and hook service workers in particuarl. 
- `mitmdump`: (recommended) [Mitmproxy](https://mitmproxy.org/)
- `mockttp`: (partial support) [Mockttp](https://github.com/httptoolkit/mockttp)
Even though we have supported `mockttp` in the past, mostly to provide an alternative to `mitmdump`, we have done most of our measurements using the latter, and recommand going with it to get the most of our analysis framework.  

## extensions
extensions to be installed on a Chromium browser when `automation` is done with `playwright` or `puppeteer`. Note that this is not equivalent to doing the automation with an extension. This option simply makes it possible for you to install an extension that implements a custom logic on top of the normal analysis being done.


## requestypes
Types of requests whose headers are to be logged. Differences exist between automation tools, i.e. [Puppeteer/Playwright](https://playwright.dev/docs/api/class-request#request-resource-type) or [WebExtensions](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/webRequest/ResourceType). When the 

## setup
- `prowsebox`: default value, passively monitors service workers events and APIs calls
- `sqwr`: enforces the [swebRequest](https://github.com/sqwr/swebrequest) security and privacy features on top of a service worker.


## Examples
1. Chrome Devtools Protocol
```json
{
    "browser": "chromium", // optional
    "automation": "playwright", // default or "puppeteer"
    "mitmproxy": "mitmdump", // remove if Chrome Devtools Protocol (CDP)

}
```