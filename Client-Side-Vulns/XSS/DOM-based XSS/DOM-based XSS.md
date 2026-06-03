---
sticker: emoji//2705
---
When Javascript **passes adversary-controlled data from an input source (like URL) to a sink that supports dynamic code execution**, such as `eval()`, `innerHTML`, `window.location` (accesses data in the URL). 
- **`SOURCE`** --> **`SINK`** (`eval(), .innerHTML`, etc.) --> *JS execution.*

Depending on the **source/sink relationship**, may be able to insert a #payload via:
- URL Query strings (`site.com/?=xxxxx&yyyyy`)
- URL fragments (`site.com/#xxxx`)
- URL **path**, within certain apps (PHP, 404 page) 
  (`site.com/PAYLOAD/`)

**Sink:** the **function** that **manipulates the DOM**, and allows execution of arbitrary Javascript. (`.innerHTML`, `document.write`, etc.)

To exploit, must analyse how **inputs/sources** can **influence how the Javascript manipulates the DOM**, and the subsequent rendering of the page content. 
- **[List of sinks that can lead to DOM manipulation](https://portswigger.net/web-security/cross-site-scripting/dom-based#which-sinks-can-lead-to-dom-xss-vulnerabilities)**
### DOM-based XSS Process - HTML Sinks
1. Search for a **unique value**—`zjhvxsa`—and analyse where it appears within the response (find the **sink**). Use **DevTools** for this (NOT **View Source** - doesn't reflect javascript changes within HTML)
2. Scan for **functions/sinks** within javascript that recieve & process **user-manipulated data** (e.g. from `window.location.search`). List of [common sinks here](https://portswigger.net/web-security/cross-site-scripting/dom-based#which-sinks-can-lead-to-dom-xss-vulnerabilities).
3. Inspect how the unique value has entered the sink. **Fuzz for any encoding/escaping/filtering**. Attempt changing the input to **break out of sink's context** (injecting `'`, etc.)
4. **If your data gets URL-encoded before being processed, an XSS attack is unlikely to work.**

### DOM-based XSS - JavaScript sinks
#dom-invader #sinks #sources
**TLDR;** use [DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader) (Burp Chrome only)
As input doesn't necessarily appear within the DOM, can be harder to follow how it flows from source to sink.
1. Search for a **unique value**—`zjhvxsa`—and analyse where it appears within the response (find the **sink**).
2. Use the **JavaScript debugger to add break points**, to follow how the source's value is used - whether its **assigned to variables**, and **where** those variables get passed (if into sinks) and **how.**
3. Once you've found **sink processing data from the source**, use the debugger to **inspect the value by hovering over the variable to show its value** before being sent to the sink. Refine this payload accordingly to attempt to execute XSS.

***Note:** Browsers will behave differently! E.g. for URL-encoding, Chrome, Firefox, and Safari will URL-encode `location.search` and `location.hash`, whereas others don't.*
### Semantics:
- `.document.write` sinks: `<script>` tags **do work**
- `.innerHTML` sinks - use alternative elements like `img` or `iframe` (`<img src=x onerror=alert()>`) *(as `<script>` tags **DON'T work**, )*

