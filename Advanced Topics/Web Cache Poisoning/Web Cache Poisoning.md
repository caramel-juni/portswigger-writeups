#web-cache #web-cache-poisoning
## What are Web Caches?
Web Caches are deployed **between the web server & client** to **store/"cache" the responses to common requests**, improving website loading times/reducing server load. When clients send equivalent requests to those already cached, the web cache **serves a copy of the cached response directly to the user**, without any interaction from the back-end.

Caches identify equivalent requests with **Cache Keys**, a **predefined subset of the request's components** (often the request line & `Host:` header). Components not included are "unkeyed". Cached responses will be **served to all requests matching the cache key** until the response expires.

![](attachments/51878.png)

All web cache poisoning requires **manipulating unkeyed inputs**, i.e. **headers not included in calculating the cache key.** This is so they can be freely injected into & manipulated **without affecting the cache key**/the request's "fingerprint", and thus **match the cache key/subsequent request served to other users.**

#### Testing for unkeyed inputs:
- Manually by **adding random inputs to requests for a unique page**, often marked with an arbitrary/non-existent query parameter called a #cache-buster (e.g. `GET /?cb=123`, a "cachebuster" request) to ensure the cached response can be tracked.
- Using **Param Miner** - for a request, right-click + select "`Guess headers`". Will run in background, sending requests containing inputs from a built-in list of headers, and flag any requests that have an effect on the response. Can add a #cache-buster automatically, too.

## Exploiting Web Cache Poisoning
Requires:
- An input to be **reflected, unsanitised** in the corresponding response, or used to **dynamically generate other data (server or client side)**
- The injected input **must not affect the cache key**, so the poisoned response's cache key will match that of "normal" requests, so it can be served to other users.
	- *E.g. Crucially for web cache poisoning, the `X-Forwarded-Host` header is often unkeyed*
However, determining **whether a request is cached** and **for how long** can be tricky and depend on several factors like:
- File extension
- Content-type
- Route of the request
- Status code
- Response headers


## ==USE PARAM MINER TO DISCOVER HIDDEN, SUPPORTED & UNKEYED HEADERS==
Right click on request --> `Extensions` --> `Param Miner` --> `Guess Headers` (& others)
- [Unofficial documentation for the option checkboxes](https://github.com/nikitastupin/param-miner-doc/blob/master/README.md)
![](attachments/Pasted%20image%2020260530181040.png)

### Practical examples within writeups:
- [Practical Web Cache Poisoning](https://portswigger.net/research/practical-web-cache-poisoning)
- [Web Cache Entanglement: Novel Pathways to Poisoning](https://portswigger.net/research/web-cache-entanglement)****


- #### [Exploiting cache design flaws](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws)
- #### [Exploiting cache implementation flaws](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws)
