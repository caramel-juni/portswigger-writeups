---
sticker: emoji//2705
---
**Content Security Policy:** browser security mechanism that restricts the resources (e.g. scripts/images) that a page can load, as well as whether it can be `iframe`d by other pages. To enable, each response must contain the HTTP header `Content-Security-Policy` with values dictating the policy. E.g.:
- `script-src 'self'` - only allows scripts to be run from the **same origin** ( #origin: same **URI scheme/protocol**, **domain** and **port number**) as the page itself.
- `script-src https://evil.com` - only allows scripts from specified domain.
*Take care when allowing external domains - as may be able to manipulate `CDNs` like `ajax.googleapis.com` to host & deliver malicious content.*

**NOTE:** `CSP`s will often block **scripts, but rarely images** - so always try to use `img` elements to make requests to external servers, in order to disclose CSRF tokens etc.

### Specifying trusted resources: Nonces & Hashes
CSP can also specify:
- **A nonce** (a random value - `jhd638t3v`) that must be **used in the tag that loads a given script** (e.g. `<script nonce="jhd638t3v"/>)`). Must be securely generated upon page load & non-guessable.
- **A hash of the trusted script** - which will refuse to run if the script's hash doesn't match that in the `CSP`.

## CSP Protections:
Browsers will often have **built-in dangling markup mitigation**, blocking requests containing certain characters, such as raw, unencoded new lines or angle brackets.

May attempt to block #dangling-markup with directives like:
- `img-src 'self'`
... but doesn't protect against `<a>` tags with a dangling `href` attribute.

- https://portswigger.net/research/evading-csp-with-dom-based-dangling-markup
#### Clickjacking protections:
To prevent pages being framed:
- `frame-ancestors 'none'` or `frame-ancestors 'self'`
*Can be combined with `X-Frame-Options` header for older browser support, but that only validates the top-level frame (wheras CSP validates **each frame in the parent frame hierarchy**)*

## CSP Evasion - Policy injection
Occasionally, can inject **into the `CSP`** with directives like `report-uri`, which is typically at the **end of the `CSP`.**
This means you'll have to **overwrite existing directives** to control the `CSP`, which usually isn't possible, except for:
- **In Chrome, can use the new `script-src-elem` directive**, which allows you to [overwrite existing `script-src` directives](https://portswigger.net/research/bypassing-csp-with-policy-injection) for **`<ELEMENTS>`** only (e.g. `<script>` tags)
- Can also try `script-src-attr` for inline script handlers (e.g. `onclick`)
- All others fall under `script-src`
### Lab: 
- Observe there's an XSS on search bar, but blocked by CSP.
![](attachments/Screenshot%202026-03-27%20at%2011.35.24%20am.png)
- Examining the CSP itself, can see: `report-uri /csp-report?token=`. This uses `token` as a parameter, see if we can inject into it to control the CSP. 
  Using the search query `.../?token=%20helloworld`, can see it reflected in CSP. Thus, we can try and **add new directives to CSP** to override strict policy. ![](attachments/Screenshot%202026-03-27%20at%2011.37.51%20am.png)
- Using the `script-src-elem`, we can [specify valid sources for javascript script elements](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/script-src-elem) (Chrome only), and allow execution of `<script>` tags.
- Based off the error messages, we understand that the `script-src-elem` requires either the `unsafe-inline'` keyword, a hash or nonce. ![](attachments/Screenshot%202026-03-27%20at%2011.55.14%20am.png)
- Thus, can adjust final payload: `; script-src-elem * 'unsafe-inline';`
``` js
/* pre-url encoding */
/?search=<script>alert(document.domain)</script>&token=; script-src-elem * 'unsafe-inline';
```