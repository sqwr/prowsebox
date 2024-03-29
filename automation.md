# Automation Clients
An automation tool or client is the party that is responsible for or mediate (with additional components) the navigating URLs in a browser (online or offline), monitor their status (success, failure), gather the service workers APIs logs, extract information from webpages DOMs, etc. There are 3 main clients currently supported:
- [CDP](#cdp): the [Chrome Devtools Protocol](https://chromedevtools.github.io/devtools-protocol/) or more specially, the [Puppeteer](https://pptr.dev/) and [Playwright](https://playwright.dev/) that are built on top of it and present an interface for automating a whole bunch of browsers, directly interacting with web pages ( and  service workers  in the case of Chromium browsers). Playwright in particular supports most desktop browsers, including Chromium, Safari and Firefox. Chrome browser on Android are also experimentally supported.
- [webExtension](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions): 


## CDP
Pros
- Automation tools maintained by big companies: Google and Microsoft and extensively used in the wild
- Push notifications permission can be automatically granted to all navigated websites upon request
- Support of offline mode
- Support for most desktop browsers: Chromium, Firefox, Webkit Safari
- Headless mode supported out of the box
- Support of direct interactions with service workers for collecting logs (Chromium browsers)

Cons
- Automation tools such as Puppeteer and Playwright can be [detected and potentially blocked](https://datadome.co/bot-management-protection/detecting-headless-chrome-puppeteer-extra-plugin-stealth/)
- Push notifications are not shown in headless mode
- Headless mode is not always equal headful mode without a window: See the [announcement of Chrome 112](https://developer.chrome.com/articles/new-headless/)
- Playwright/Puppeteer are only guaranteed to work with the bundled browser versions they are tested on


## webExtension
Pros:
- Work with full fledged headed browsers: more difficult to detect by bot detectors
- Can collect service workers APIs logs occurring without actively navigated pages (i.e. push notifications)
- Setup can be fully automated

Cons:
- Headed browsers may use more resources
- Offline mode is not supported with extensions
- Works only for browsers supporting the webExtensions API: desktop Chromium, Kiwi Browser on Android, desktop Firefox and Firefox on Android
- Granting automatically the notification permissions to all websites is not possible with Firefox




## webPage
This automation method has been added mostly to support mobile browsers on iOS, as none of the other methods work on this platform. Nonetheless, this method works for most browsers on all plaforms, provided that they accept the custom certificate authority (CA) of the Mitmproxy server.

In a nutshell:
1. Configure the 

Pros:
- Works with all browsers, 
- Can be used to test 

Cons:
- Require manual interactions to start the navigation of URLs (in particular on mobile browsers). 
  - On the Desktop br
  - On Android, one can leverage tools like [ADB](https://developer.android.com/studio/command-line/adb) to automate the navigation of URLs in browsers. 
- Notifications permissions cannot be automatically granted to navigated websites
- Require manual interactions for the setup on mobile devices:
  - installing the custom CA on the mobile devices
- Offline mode is not supported

## Mitmproxy

##