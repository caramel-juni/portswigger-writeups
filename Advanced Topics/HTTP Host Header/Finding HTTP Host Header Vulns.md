---
sticker: emoji//2705
---
## How to find `Host:` Header vulnerabilities:

#### In short:
- **Supply arbitrary `Host:` headers** & test whether app is still accessible *(as Burp allows this by maintaining separation between `Host:` header and IP address - which is derived from the `Target URl` in Repeater, for example.)*
- **Inject host override headers** with malicious input - e.g. 
	- `X-Forwarded-Host` + `badstuff`
	- `X-Host`
	- `X-Forwarded-Server`
	- `X-HTTP-Host-Override`
	- `Forwarded`
	Because many sites are accessed via an intermediary/proxy system, the resulting `Host:` header the backend recieves **may contain the domain name for one of these intermediary systems**, which is not useful. So, the front end may inject the `X-Forwarded-Host` header, containing the **original `Host:` header from the client's initial request**. This is often supported by frameworks **even if not used**, and thus supplying an arbitrary `X-Forwarded-Host:` may take precedence and can be used to inject malicious input.
	- *Use the [Param Miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943) extension's "Guess headers" function to automatically probe for supported headers using its extensive built-in wordlist.*
- Try several headers with variations on the `localhost` IP, **as well as other internal IPs/metadata IPs based on the known tech stack.**
``` bash
# VALUES
localhost
127.0.0.1
127.1
## Use alternative representations of 127.0.0.1:
2130706433
017700000001

# HEADERS
Host:
	Host:
host:
X-Forwarded-For:
X-Real-IP:
CF-Connecting-IP:
Client-IP:
X-Host:
X-Forwarded-Server:
X-HTTP-Host-Override:
Forwarded:

```
- **Attempt to access other #virtual-hosts at the same IP**: can **brute-force a list of potential internal subdomains** (found via info disclosure or the like) such as `Host: intranet.example.com`, `Host: test.site.com`, `Host: dev.site.com` etc. Can typically access any of these virtual hosts on the server with `Host: [vhost.domain]`, even though internal sites often don't have public DNS records & are thus not easily publicly routable via public IP. 
- **Check for flawed `Host:` validation** - inject into the port field (e.g. `Host: vulnerable.com:bad-stuff`), prefix the permitted `Host:` value with text to check for arbitrary subdomains (e.g. `Host: totallynotvulnerable.com`), use a subdomain you control (e.g. `Host: totallynot.vulnerable.com`)
- **Send ambiguous requests:**
	- Duplicate Host headers (where front/backends treat them differently, e.g. frontend routes to **first** `Host:` but backend parses **both of them/the other one**)
	- **Supply an absolute URL** (test HTTP & HTTPS) *and* a `Host:` header: 
	  `GET https://vulnerable-website.com/ HTTP/1.1`
	  `Host: bad-stuff-here`
	- **Indent duplicate headers** with a space/tab character (some servers will interpret this as a wrapped line/the previous header, or ignore it entirely)
	- **Adapt HTTP request smuggling techniques** ([link](https://portswigger.net/web-security/request-smuggling))
- Try a [connection state attack](Exploiting%20HTTP%20Host%20Header%20Vulns.md#Connection%20state%20attacks) by **sending requests in a grouped sequence** (`Host:` validation may only be performed on the first request)
- Attempt [Web Cache Poisoning](../Web%20Cache%20Poisoning/Web%20Cache%20Poisoning.md) using a cachebuster `/?cb=123` for any valid `Host:` reflection discovered ([Lab: Web cache poisoning via the Host header](Exploiting%20HTTP%20Host%20Header%20Vulns.md#Web%20cache%20poisoning%20via%20the%20Host%20header))
- Attempt [Dangling-markup Attacks](../../Server-Side%20Vulns/Authentication/Alternative%20Authentication%20forms.md#Dangling-markup%20Attacks) ([Burp Link](https://portswigger.net/web-security/cross-site-scripting/dangling-markup)) if the `Host:` is used in any dynamic HTML generation/for a password reset sent to email (e.g. a `Click Here`, using host-header-created `<a>` link, to inject into & leak rest of email if contents are scanned by AV or similar - see [linked lab](../../Server-Side%20Vulns/Authentication/Alternative%20Authentication%20forms.md#Dangling-markup%20Attacks).)
*[Can read these in full, here](https://portswigger.net/web-security/host-header/exploiting#how-to-test-for-vulnerabilities-using-the-http-host-header)*



