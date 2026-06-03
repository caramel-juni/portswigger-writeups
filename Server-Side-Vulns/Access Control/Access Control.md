---
sticker: emoji//2705
---

Application of constraints on who or what is authorized to perform actions or access resources.

- **Authentication:** Confirms a user is **who they say they are**
- **Session Management:** identifies which subsequent HTTP requests are being made by the same user
- **Access Control/Authorization:** determines **what actions** a user is **allowed to carry out.**

## Vertical Privilege Escalation:

Gaining access to features/resources belonging to a more privileged user.

#### Unprotected Functionality:
E.g via force-browsing to potentially sensitive pages, like `/admin` panels. 
**Done via:**
- Guessing directory names
- **Burp Passive scan** --> crawling
- Right click on sitemap branch --> Engagement Tools --> **Discover Content**
- `Gobuster`/`Ffuff`/ `Ferroxbuster`
- Scraping locations of privileged files from other locations (`/robots.txt`, comments)
- Analysing `javascript` references - do a site-wide search in burp for common terms "admin", etc.)
``` html
<script> 
	var isAdmin = false; 
	if (isAdmin) {... 
			var adminPanelTag = document.createElement('a'); 
			adminPanelTag.setAttribute('href', 'https://site.com/admin-panel-yb556'); 
			adminPanelTag.innerText = 'Admin panel'; ...} 
</script>
```

#### Parameter-based access controls:
Some apps determine rights at login, and store in:
- A hidden field
- A cookie (`Role=Admin`)
- A preset query string parameter (e.g. `https://site.com/login/home.jsp?role=1`)
##### Techniques:
- Can attempt to **perform a state-changing action** (like email change) and use it to change other parameters (like `roleid` or `admin=true` etc.) ![](attachments/Screenshot%202026-03-15%20at%202.58.50%20pm.png)
- Use **non-standard HTTP headers** like `X-Original-URL` and `X-Rewrite-URL` to redirect & circumvent strict **front-end access controls** (e.g. restricting `POST` to URL `/admin/deleteUser`, can `POST` to `/` and redirect with `X-Original-URL: /admin/deleteUser`).
  e.g.:![](attachments/Screenshot%202026-03-15%20at%203.08.08%20pm.png)
- Use **different [HTTP Methods](https://www.geeksforgeeks.org/java/what-is-the-difference-between-put-post-and-patch-in-restful-api/)** to update data at restricted URLs (e.g. `GET`/`PATCH` when `POST` is expected)
  ![](attachments/Screenshot%202026-03-15%20at%203.44.47%20pm.png)
- **Abuse URL-matching discrepancies:** e.g:
	- **Appending `/` to endpoints** (`/admin/deleteUser` vs. `/admin/deleteUser/`)
	- **Arbitrary Suffixes** - `/admin/deleteUser.anything` --> redirects to `/admin/deleteUser` (with `useSuffixPatternMatch` in #SpringFramework)
	- **Inconsistent capitalization** (`/ADMIN/DELETEUSER` --> `/admin/deleteUser`)
---

## Horizontal Privilege Escalation:
If a user is able to gain **access to resources belonging to another user** of the same privilege type.
#### Techniques:
- Modifying parameters in URLs/requests (e.g. `id=X` --> an `IDOR` vulnerability)
	May not always be as simple as incrementing, and use `GUIDs` instead (like `796345938974-9786438-444568`). In this case, **search the app for where `GUIDs` are disclosed - e.g. reviews, user messages, etc.** 
	- Try a `regex` search for `GUID` formatting type
- **Modify values in Repeater - to analyse info leakage in redirects** before they automatically happen in browser.

## [[[Portswigger Academy/Burpsuite/Knowledge Base/Server-Side Vulns/IDOR.md|Insecure Direct Object References]]
#IDOR 
A subcategory of access control issues - if an application uses **user-supplied input to access objects directly**, can be modified to obtain unauthorized access.

- User controlled input (e.g. customer number) could be used as **record indexes in queries that are performed on the back-end database**: e.g. `https://insecure-website.com/customer_account?customer_number=132355`
- Sensitive info may be stored as **static files on the server-side filesystem** (e.g. `log1.txt`, `log2.txt`)

---

### Access control vulnerabilities in multi-step processes

Where certain functions require multiple steps, **ACLs may not be applied equally to all of them.** Always attempt to **perform the steps in isolation** after determining the flow.
*E.g. to update user details:*
1. Load the form that contains details for a specific user.
2. Submit the changes.
3. Review the changes and confirm.

Sometimes **ACLs are based on the `Referer` header** submitted in the HTTP request (added by browsers to indicate which page initiated a request).
- e.g. for subpages like `/admin/deleteuser`, sometimes if the `Referer` header contains the main `/admin` URL, will bypass ACLs.

Some websites enforce access controls based on the **user's geographical location** - use **VPNs, geographical proxies or manipulating client-side location parameters.**
