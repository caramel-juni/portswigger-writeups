---
sticker: emoji//2611-fe0f
---
#XSS: manipulating a vulnerable web site to return/execute malicious JavaScript in the context of a user's browser session, often compromising their interaction with the application. 
**Can be used for:** impersonation, carrying out actions on user's behalf, exfiltrating data or credentials, virtual defacement, injecting trojan functionality.

**Can be tested with:**
- `<script>alert(document.domain)</script>`
- `<script>print(document.domain)</script>` *(in Chrome post 2021, [cross-origin iframes are prevented](https://portswigger.net/research/alert-is-dead-long-live-print) from calling `alert()` - hence, try `print()`)*


## Types:
- [Reflected XSS](Reflected%20XSS.md) - where the malicious script **comes from the current HTTP request**, and unsafely **includes that data within the immediate response**.
- [Stored XSS](Stored%20XSS.md) - where the malicious script **comes from the website's database** (submitted in all kinds of ways: HTTP request, SMTP messages, displaying network traffic; any time data is retrieved, processed, stored & displayed by application.)
- [DOM-based XSS](DOM-based%20XSS/DOM-based%20XSS.md) - where **vulnerability exists in client-side code** (rather than server-side), involving Javscript processing data from an untrusted source in an **unsafe way** (`innerHTML`, `document.write`, etc.), usually by writing the data back to the DOM. 
	``` javascript
	var search = document.getElementById('search').value;
	var results = document.getElementById('results');
	results.innerHTML = 'You searched for: ' + search;
	```

### Identifying XSS Vulnerabilities:
- Submitting simple, unique inputs (such as a short alphanumeric string, `zhsgv`) into every entry point in the application, checking where it appears (`sinks`)
- Identifying how & where the submitted input is returned in HTTP responses.
- Testing each location individually to determine whether suitably crafted input can be used to execute arbitrary JavaScript
**DOM-based in non-URL-based (such as `document.cookie`) or non-HTML-based sinks (like `setTimeout`):**
- Must manually review JS code, & attempt to interact with it first in the browser, then in the application.

### Related components/attacks
- [Content security policy](https://portswigger.net/web-security/cross-site-scripting/content-security-policy) #CSP 
- [Dangling-markup Attacks](../../Server-Side%20Vulns/Authentication/Alternative%20Authentication%20forms.md#Dangling-markup%20Attacks) ([Burp Link](https://portswigger.net/web-security/cross-site-scripting/dangling-markup))

### Preventing XSS:
- **Strictly filter user input on arrival.**
- **Encode data in HTTP responses upon output.** (may require combos of HTML, URL, JavaScript, and CSS encoding)
- **Use appropriate response headers** in responses not intended to contain any HTML/JS (use `Content-Type` and `X-Content-Type-Options` headers)
- Set a strict **Content Security Policy.**

