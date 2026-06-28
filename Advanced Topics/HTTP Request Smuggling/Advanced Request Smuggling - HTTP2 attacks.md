---
sticker: emoji//2757
---
#http2 
- [HTTP/2: The Sequel Is Always Worse](https://portswigger.net/research/http2)

Whilst it's generally recommended to implement HTTP/2, it can actually make some websites **more** vulnerable to request smuggling, as opposed to less, opening up:
- New [attack vectors for request smuggling](https://portswigger.net/web-security/request-smuggling/advanced#http-2-request-smuggling)
- The ability to [persistently poison the response queue](https://portswigger.net/web-security/request-smuggling/advanced/response-queue-poisoning)
- Construct exploits even where [the target doesn't reuse the connection](https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling) between the frontend & backend servers

## HTTP/2
- Utilises a **single mechanism** for determining message length
- Messages sent as a **series of "frames"**, each with a **specific length field** specifying **how many bytes the server should read**.
	- Thus, the **`request length = sum of frame lengths`**
So, in theory, the message length ambiguity needed for request smuggling is removed... **unless the websites do not use HTTP/2 end-to-end**, which can be achieved via #http-downgrading.

### HTTP Downgrading
Rewriting HTTP/2 requests **using HTTP/1 syntax**, generating an equivalent HTTP/1 request, often to communicate to legacy backend servers that only support HTTP/1.
- **Use Case:** Often performed by web servers/reverse proxies to allow clients to communicate with HTTP/2 while supporting **backend servers that only use HTTP/1**.
  ![](attachments/Pasted%20image%2020260622105141.png)
- The downgraded request is converted & sent to the client/backend server by the reverse proxy/webserver, and often is **on by default** or **unable to be disabled**.
  ![](attachments/Pasted%20image%2020260622105209.png)
***Note:*** *As HTTP/2 is a binary protocol, the presentation is simplified here: messages aren't broken into "frames", headers use plaintext and pseudo-headers are distinguished by a colon. Will look different OTW.*

Enables several kinds of HTTP/2 downgrading request smuggling attack vectors (aka different ways to specify & abuse request length):
- `H2.TE` & `H2.CE`
- Response queue poisoning
- Request smuggling via CRLF injection
- HTTP/2 request splitting
- HTTP request tunnelling
- 0.CL request smuggling

### H2.TE
As HTTP/2 requests don't *have* to specify length in the header, during downgrading a `Content-Length:` will be **added by the intermediary server**, its value derived from [HTTP/2's built-in frame-sum length mechanism](https://portswigger.net/web-security/request-smuggling/advanced#http-2-message-length). However, **HTTP/2 requests can contain their OWN `Content-Length` header**, and this may be **reused in the resulting rewritten HTTP/1 request.**
While the `Content-Length` header ***should*** match the calculated HTTP/2 in-built message length, validation is not always performed properly before downgrading, and so:
- Front-end uses **implicit HTTP/2 request length** to determine where request ends
- Back-end **refers to forged `Content-Length` header**, resulting in a desync.

e.g. **Front-end:**
**HTTP/2:**
![](attachments/Pasted%20image%2020260622110938.png)
**HTTP/1.1 rewrite received by backend:**
``` http
POST /example HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

GET /admin HTTP/1.1
Host: vulnerable-website.com
Content-Length: 10

x=1GET / H
```
You may also want victim's headers appended to the smuggled prefix (cookies, etc.), but these can interfere with certain attacks (duplicate headers). 
To prevent this:
- Include a trailing parameter, e.g. `x=1`
- Include a longer `Content-Length` header in the smuggled request
Thus, victim's request will be included but **truncated before the headers.**

##### Lab: [H2.CL request smuggling](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-cl-request-smuggling)
**Objective:** *This lab is vulnerable to request smuggling because the front-end server downgrades HTTP/2 requests even if they have an ambiguous length.
To solve the lab, perform a request smuggling attack that causes the victim's browser to load and execute a malicious JavaScript file from the exploit server, calling `alert(document.cookie)`. The victim user accesses the home page every 10 seconds.*

Page seems to load& use a tracking script, loading it as arbitrary javascript. Likely means that we need to perform a Host rewrite to load this from our server, to execute arbitrary js. 

*Could not get this to work after trying for ages, with the plugins + intruder for request timing, to no avail.*
*I'm leaving HTTP request smuggling for now, and coming back later.*