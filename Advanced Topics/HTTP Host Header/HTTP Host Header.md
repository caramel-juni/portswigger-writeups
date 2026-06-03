---
sticker: emoji//2705
---
## What is the HTTP `Host:` Header?
A mandatory HTTP request header, **specifying the domain name** or **identify which back-end component** the client wants to communicate with. Used when a single IP address (resolved from the URL) might be hosting **multiple websites and applications**, in the case of:
- **Intermediary Router** - i.e a CDN or Load Balancer
- **Virtual Hosts** - where **several websites with different domains** are accessible under the **same shared server IP**.
In both of these scenarios, the `Host:` header is relied on to specify the intended recipient.
#### Steps:
1. User visits `https://portswigger.net/web-security`
2. URL resolves to the IP address of a particular server.
3. When this server receives the request, it checks the `Host:` header to determine the intended back-end (`portswigger.net`), and forwards the request accordingly:
``` http
GET /web-security HTTP/1.1 
Host: portswigger.net
```

==This can be manipulated if the server implicitly trusts the `Host:` header, or fails to validate/escape it properly! ==
This is because the `Host:` header is **often used in subsequent requests/retrieved and parsed by the application in some way**, such as when generating an absolute URL or similar. It can also be manipulated & create parsing discrepancies when additional HTTP headers, like `X-Forwarded-Host`, are supplied. 
This can lead to:
- Web cache poisoning
- Business logic flaws in specific functionality
- Routing-based SSRF
- Classic server-side vulnerabilities, such as SQL injection


