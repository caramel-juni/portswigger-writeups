---
sticker: emoji//2705
---
#path-traversal
Read/write files running on the server running the application, such as:
- Application code and data.
- Credentials for back-end systems.
- Sensitive operating system files.

To return a resource, applications append the requested filename to a specified directory (typically within the webroot directory, e.g. serving images from `/var/www/images/`) and uses a filesystem API to retrieve the contents.

Thus, can traverse "up" directories and retrieve OS files with:
- `../` (Linux): `https://insecure-website.com/loadImage?filename=../../../etc/passwd`
- Both `../` and `..\` on Windows: `https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini`

## Path Traversal Defence Bypasses:
In cases where an app may strip/block directory traversal sequences, can attempt the following...
- Absolute Path (`filename=/etc/passwd`)
- Nested traversal sequences, such as `....//` or `....\/`.
- Encoding/double-encoding traversal sequences (`%2e%2e%2f` & `%252e%252e%252f`)
- Discern & include the value of the full path (e.g. `/var/www/images`) in traversal attacks
- Use a null byte (`%00`) to terminate the file path before any required extension, e.g. `filename=../../../etc/passwd%00.png`

**NOTE:** Remember - even if something is successfully stripped **once**, try adding **multiple** in case it's not consistently applied!

### Absolute Path bypasses:
Using an **absolute path from the filesystem root**, such as `filename=/etc/passwd`, to directly reference files.
**![](attachments/Pasted%20image%2020260403173844.png)**
### Nested traversal sequences
Try nested traversal sequences (`....//` or `....\/`). These normalise to `../` but can bypass simple simple character stripping filters for `../`.
![](attachments/Path%20Traversal.png)

### (double) URL encoding `../`
Where sequences are being stripped from the URL path or the `filename` parameter of a `multipart/form-data` request, try **URL (or other non-standard encodings) encoding the `../`.** Use a wordlist for this (like Burp's inbuilt one).
![](attachments/Path%20Traversal-1.png)

### Validation of start of path *(e.g. `/var/www/images`*
Discern & include the value of the full path in traversal attacks
![](attachments/Path%20Traversal-2.png)

### Requirement to end with a valid file extension (`.png`)
Where apps may require the user-supplied filename to **end with an expected file extension**, such as `.png` - use a null-byte (`%00`) to terminate the file path before the required extension, e.g. `filename=../../../etc/passwd%00.png`
![](attachments/Path%20Traversal-3.png)