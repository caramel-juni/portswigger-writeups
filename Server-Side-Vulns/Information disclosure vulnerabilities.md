---
sticker: emoji//2705
---
**Information disclosure:** when a site unknowingly reveals sensitive information to users, typically after an adversary interacts with the site/functionality in unintended ways. Might include: 
- Non-public user attributes (user ID, password, email, payment/address details), & #IDOR vulnerabilities
- Sensitive/commercial/classified business data
- **Technical information** about the site/backend & infrastructure
- Backend **secrets, API keys**, **endpoints**, **IPs**, **connection strings**, etc.
- **Debug pages:** hostnames, server files/directory names, session variables/parameters to manipulate, hardcoded keys etc. (`phpinfo.php`, `*.sql` etc. - *use a wordlist!*)
- Application **directory structure** (`robots.txt`, `sitemap.xml`)
- **Directory listing** pages
- **Source code**/**backup files**, verbose/revealing **comments in code**
- Overly **verbose/descriptive error messages** - reveal info on:
	- Framework & technology
	- Version info
	- Parameters/objects to access/inject into
	- Payload-crafting feedback
	- *If site is open source, & you have time - check the source code for any hidden/useful functionality!*
- **Differences in application behaviour** hinting at **presence/absence of resources**

**Severity depends highly on context**, and the **impact** and **exploitability** of the leaked information - *what could an adversary **do** with it?* (e.g. follow-up attacks, known CVEs, etc.)

## Discovery Techniques:
- **Fuzzing**: identify interesting parameters, and:
	- **Active scan** branches/requests where safe to do so
	- **Use Burp Intruder** to fuzz specific injection points
	- Use **`grep`/matching rules** & project-wide search to **extract & search for keywords**
	- **`Sensitive Discoverer`** extension
	- Compare **HTTP status codes**, **response times** & **lengths**
- **Engagement Tools:**
	- **`Content Discoverer`:** identify, using brute-force wordlists & link crawling, additional content and functionality not explicitly linked on the visible site
	- **`Find Comments/Scripts`** feature - auto-extracts comments/scripts from requests to review
	- **`Burp Search`** - supports regex, negative search
![](attachments/Pasted%20image%2020260605191011.png)

#### Changing request to expose source code:
As most code is executed server side with the output sent to the browser, can't always access **raw code-based contents** of files as text (e.g. `home.php`). **However**, when files are beign edited, **temporary backup files** are often generated, often denoted by appending a tilde (`~` or `.`) to the filename or using a different extension (`.tmp`). 
Search for **variations of the target filename** (targeting it to the specific language/framework as well), like:
- `~home.php`
- `.home.php`
- `home.tmp`

#### Insecure configurations - the HTTP `TRACE` method
The HTTP `TRACE` method is diagnostic - will **echo the received request in the response**. This can occasionally **leak names of internal authentication headers/values** that may be appended to requests by reverse proxies.

#### Version control history - `/.git` folders/files
Occasionally, websites may disclose version history. 
- Check the repository & commit history, if open source
- Scan (discover content) for `.git` and `/.git` folders
- Trufflehog/similar scanning programs
Can then **download `/.git` & browse with local `git` installation**, can read/analyse `git diff`s to see snippets of code, comments, hardcoded data, removed keys etc.
- `wget -r "https://site.com/.git"` (linux only - use WSL in windows)
- `git log --oneline` - check commit history
- `git checkout [COMMIT-ID] .` - revert working directory to past commit version

---

#### Lab: [Information disclosure in error messages](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-error-messages)
- Browsing the lab until identified a request with a parameter, attempted to submit malformed requests. `500` error reveals framework:
  ![](attachments/Information%20disclosure%20vulnerabilities.png)

#### Lab: [Information disclosure on debug page](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-on-debug-page)****
Ran `Discover Content` - found `/cgi-bin/phpinfo.php`, containing `SECRET_KEY`
  ![](attachments/Pasted%20image%2020260605192740.png)

#### Lab: [Source code disclosure via backup files](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-via-backup-files)
- Ran `Discover Content` - found `/backup`, which held the source code & hardcoded password in `ProductTemplate.java.bak`:
  ![](attachments/Pasted%20image%2020260605192315.png)
- Also scanned the site map after exploring site for comments, scripts, etc. 



#### Lab: [Authentication bypass via information disclosure](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-authentication-bypass)
**Objective:** *This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end. To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete the user `carlos`.*

Using the info in the objective, changed the `POST` login request (or any page) to a HTTP `TRACE` to reflect the request & **custom headers** in the response:
![](attachments/Information%20disclosure%20vulnerabilities-3.png)
Using this header, and changing the value from your own reflected IP to something like `localhost` or `127.0.0.1`, reveals the admin panel:
![](attachments/Pasted%20image%2020260605195600.png)

Set this as the custom header via `Session Handling` rules. Ensure to set the `Scope` to `Proxy` to have it auto-included when browsing. Then, can reload page, check it's being added properly in `Logger`, & access admin panel!
![](attachments/Information%20disclosure%20vulnerabilities-2.png)
![](attachments/Pasted%20image%2020260605200038.png)



#### Lab: [Information disclosure in version control history](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-version-control-history)
**Objective:** *This lab discloses sensitive information via its version control history. To solve the lab, obtain the password for the `administrator` user then log in and delete the user `carlos`.*

- Ran `Discover Content` - found `/.git`, downloaded it:
- Browsing it, found interesting comment: `0a1fd3c3ef9c7ad1f7c4e208b8aa45bac36e2ab3 b60a2853da044c48a25308b69722cd94231ba6c4 Carlos Montoya <carlos@carlos-montoya.net> 1780727302 +0000	commit: Remove admin password from config`
- Likely want to revert to this version. Downloading the whole `.git` with `wget` inside WSL, we can then open it and browse previous commits. We want to revert to the commit before the admin panel password was removed. Can do this by moving into the folder with the `/.git` directory, running `git log --outline`, and then reverting to the previous commit with `git checkout [FIRST-COMMIT-ID]`. Then, we read `admin.conf` for the password!
  ![](attachments/Information%20disclosure%20vulnerabilities-4.png)

---

