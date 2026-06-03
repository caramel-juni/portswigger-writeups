## [Vulnerabilities in password-based authentication:](https://portswigger.net/web-security/authentication/password-based)
### Attack Types:
- **Brute force** (`hydra`, burp intruder) --> **tailor wordlists** to a given target
	- Check whether usernames are displayed publicly
	- Profile/login name collisions
	- Common email formats/patterns
	- Check HTTP responses for disclosed emails/use extensions like **[Sensitive Discoverer](https://github.com/portswigger/sensitive-discoverer)** in burp
	- For passwords, **consider human behaviour & laziness** - letter/number substitutes, iterating on passwords, password reuse, OSINT info reuse
	- **Username enumeration** - generate shortlists of valid usernames if responses differ based on whether an account exists

### Analysing differences in responses
Consider differences in **status codes, error messages, and response times** when generating a password/username shortlist.
- Ensure to extract changes in sections of the response body with `grep`, to analyse subtle response differences *(even just a missing `.` or missing error message could indicate an account/resource exists!)* ![](attachments/Screenshot%202026-03-16%20at%202.03.30%20pm.png)![](attachments/20779%201.png)
- **Check timing of requests/responses** - enable `Response received`/`Response Completed`, **set the password to a very long string of characters**, then look for any discrepancies/delays. ![](attachments/66573.png)
### Bypassing brute-force protection
Brute-force protections typically are:
- IP-based blocking
- Account lockouts
- Rate Limiting
##### Bypassing IP-based blocking:
- If site supports `X-Forwarded-For` header, try different IPs/numbers there (can iterate with **Intruder Pitchfork** Attack)
- **At regular intervals, include your own correct login credentials** throughout the wordlist, to "unblock" IPs.
	- Use TurboIntruder for this, ensuring to set to one request at a time with `concurrentConnections=1`, & `requestsPerConnection=1`. ![](attachments/Screenshot%202026-03-16%20at%207.03.13%20pm.png)![](attachments/Screenshot%202026-03-16%20at%207.02.43%20pm.png)
- **Issue multiple identical requests to identify account lockouts** and potentially **enumerate usernames** with a **Clusterbomb Attack** (all combinations of each payload set) + **Null Payloads**: ![](attachments/40263.png)![](attachments/48969.png)

#### Bypass Rate Limiting: Issuing multiple logins per request
Typically, the offending IP can only be unblocked in one of the following ways:
- Automatically after a certain period of time has elapsed
- Manually by an administrator
- Manually by the user after successfully completing a CAPTCHA

However, can attempt to manipulate their apparent IP (e.g. `X-Forwarded-For`) or [issue **multiple logins per request**](https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request) to bypass this. E.g. if the site accepts JSON, attempt to submit multiple passwords **as an `array []`**:
![](attachments/56855.png)

---

### HTTP `BASIC` Auth:
Generally **insecure**, because:
- Involves **repeatedly sending the user's login credentials** with every request (could be captured in MiTM unless the website also implements HSTS)
- As **token consists exclusively of static values**, more easily brute-forcible & vulnerable to session-related exploits like #CSRF attacks.