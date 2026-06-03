---
sticker: emoji//2705
---
Some apps may try and **use the HTTP `Referer` header** to "**verify the request originated from the application's own domain**". 
This header is auto-added by browsers, and contains the URL of the web page that linked to the resource.

### Bypass by stripping `Referer` header
#referer-stripping
Can coerce the browser into **not setting/dropping the `Referer` header with any requests**, via a method like including the `<meta name="referrer" content="never">` tag on your exploit page.

#### [Lab](https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-depends-on-header-being-present): CSRF where Referer validation depends on header being present
Can try and issue CSRF email change from the exploit server (using CSRF POC generator), but get the error `Invalid Referer Header`:
![](attachments/Bypassing%20Referer-based%20CSRF%20Defences.png)
However, when repeating the same request and **removing the `Referer` header**, the email change is successful:
![](attachments/Pasted%20image%2020260411192255.png)
Thus, if our exploit page strips the `Referer` header before sending the request by adding the `<meta name="referrer" content="never">` into the POC CSRF exploit HTML, can potentially trigger the exploit:
E.g. can see below that after visiting `/exploit` page, the `Referer` header is never sent in the subsequent request, and thus bypasses CSRF `Referer` header protection as it's absent.
![](attachments/Pasted%20image%2020260411192526.png)

---
### Bypassing insecure `Referer` validation - e.g. checking for a trusted domain/string
#referer-validation-bypass
If the app **validates the header in an insecure way**, like checking that the `Referer` URL:
- **Starts with an expected value** --> place known good domain as a **subdomain of attacker's website**
  `http://`**`vulnerable-website.com`**`.attacker-website.com/csrf-attack`
- **Contains its own domain name** --> place known-good domain name as URL query string somewhere in `Referer` URL
  `http://attacker-website.com/csrf-attack`**`?vulnerable-website.com`**
  ***Note:** However, many browsers now **strip the query string from the `Referer` header by default**.* To avoid this, ensure your POC page sets the `Referrer-Policy: unsafe-url` header! (`Referrer` spelt *correctly* this time!), e.g. via `<meta name="referrer" content="unsafe-url">`

#### [Lab](https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-broken): CSRF with broken `Referer` validation
When attempting CSRF normally, can see it's blocked due to the error `Invalid Referer Header`. 
After adding the trusted domain as a query parameter in `Referer` header, can see it goes through, suggesting the validation check is just looking for the domain *somewhere* in the `Referer` header value.
![](attachments/Pasted%20image%2020260411193750.png)
To **add this trusted domain as a URL query string (`?xxxx.com`) to the `Referer` header** in the **request sent from the CSRF POC page**, add this injected trusted string to the `history.pushState()` function of the auto-submit `<script>` on the page. This will append the trusted domain on sending:
![](attachments/Bypassing%20Referer-based%20CSRF%20Defences-1.png)

To **ensure the browser will respect this query string inside the `Referer` header** (as by default it's stripped for security reasons), also set the `<meta name="referrer" content="unsafe-url">` tag on your CSRF POC page:
![](attachments/Bypassing%20Referer-based%20CSRF%20Defences-2.png)

- Detailed Explanation: https://www.youtube.com/watch?v=N8Hjx3kCc-g

