All dynamic websites are composed of APIs (Application Programming Interfaces), so many classic web vulnerabilities (like SQL injection) could be classed as API testing.
### Helpful plugins:
- [Backslash Powered Scanner](https://portswigger.net/bappstore/9cff8c55432a45808432e26dbb2b41d8) (identify server-side injection vulnerabilities)
- [JS Link Finder](https://portswigger.net/bappstore/0e61c786db0c4ac787a08c4516d52ccf) (find API references in JS)
- [Param miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943) (guess potential parameters, e.g. `/api/user/{param-name}/`)
- [OpenAPI Parser](https://portswigger.net/bappstore/6bf7574b632847faaaa4eb5e42f1757c)

## API Recon
- Crawl with Burp [Content discovery](https://portswigger.net/burp/documentation/desktop/tools/engagement-tools/content-discovery) tool
- Use Burp **Intruder + a custom wordlist** - tailored to your app's content, based off API patterns observed when browsing site + standardised conventions *(e.g. `/api/`, try `v1` etc. variants, try other common functions like `delete` and `add`.)*
- Use **Kiterunner**
- **Search within JavaScript files** - can reference endpoints not directly triggered already.
	- [JS Link Finder](https://portswigger.net/bappstore/0e61c786db0c4ac787a08c4516d52ccf) BApp
- Use [Param miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943) - automatically guess up to 65,536 param names per request, based on info taken from the scope.

### Speaking the Language:
Must determine how to interact with the API (using docs or poking at endpoints)
- The **input data the API processes** (both compulsory and optional parameters)
- **Supported HTTP methods** and media formats
- **Rate limits** and **authentication mechanisms**.
### Documentation:
- May be **publicly available**
- If not public, **test common documentation endpoints**: - `/api`, `/swagger/index.html`, `/openapi.json`
- For all identified endpoints, **investigate each component of the path** - e.g. for `/api/swagger/v1/users/123` try `/api/swagger/`, `/api` etc.

### Tooling:
- **Burp Scanner** - crawl and audit OpenAPI docs/any in JSON or YAML.
- [OpenAPI Parser](https://portswigger.net/bappstore/6bf7574b632847faaaa4eb5e42f1757c) BApp.
- [Postman](https://www.postman.com/), Bruno, or [SoapUI](https://www.soapui.org/).

## HTTP Methods:
Can be:
- **Idempotent** - multiple identical requests yield the same results ().
- **Safe** - does not alter server state (read-only operation).
- **Cacheable** - allows clients to reuse previously-fetched responses.

**Identify allowed methods** for an endpoint by sending the `OPTIONS` method (e.g. `OPTIONS /api/products/1/price HTTP/2 ...`).

#### Identify & exploit different Content-Types
Modify the `Content-Type` header, then reformat the request body accordingly (can use [Content type converter](https://portswigger.net/bappstore/db57ecbe2cb7446292a94aa6181c9278) BApp). Can help:
- **Trigger errors** that disclose useful information.
- **Bypass flawed defences/differences in processing logic** (e.g. API may securely handle `JSON` data but be susceptible to `XML` injection attacks)

| **Method**  | **Description**                                                                    | **Idempotent** | **Safe** | **Cacheable** |
| ----------- | ---------------------------------------------------------------------------------- | -------------- | -------- | ------------- |
| **GET**     | Retrieves data from the server.                                                    | Yes            | Yes      | Yes           |
| **POST**    | Submits data to be processed on the server, often resulting in a state change.     | No             | No       | No            |
| **PUT**     | Updates or replaces a resource entirely.                                           | Yes            | No       | No            |
| **DELETE**  | Deletes a specified resource.                                                      | Yes            | No       | No            |
| **HEAD**    | Same as `GET` but retrieves only headers (no body). Used to check resource status. | Yes            | Yes      | Yes           |
| **OPTIONS** | Describes communication options available for the target resource.                 | Yes            | Yes      | No            |
| **PATCH**   | Partially updates a resource.                                                      | No             | No       | No            |
| **CONNECT** | Establishes a network connection tunnel, typically for SSL encryption.             | No             | No       | No            |
| **TRACE**   | Performs a message loop-back test for debugging.                                   | Yes            | Yes      | No            |


## Mass assignment vulnerabilities
Occurs when software frameworks **automatically bind request parameters to fields** on an **internal object**.
May result in an app supporting parameters **not intended to be processed**.

E.g. even if a `PATCH /api/users/` request typically supplies changing `name` and `email`, a `GET /api/users/123` request may reveal additional parameters - **always try and change whichever ones you can**, using both **valid & invalid options** *(as may be able to infer whether values are being used/validated in internal queries)*
``` json
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
```

#### Identifying hidden parameters:
#hidden-api-parameters
- Querying an endpoint's parameters with `GET` (which may reveal hidden parameters bound to the internal user object), then can attempt to modify them with `PATCH` or `POST`
E.g. find `chosen_discount`, check which methods are allowed with `OPTIONS`, and change `chosen_discount percentage:100` & send it.
![](attachments/Screenshot%202026-03-09%20at%205.23.28%20pm.png)

---

