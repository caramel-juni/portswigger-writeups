
here lie all my notes and lab writeups for portswigger's wonderful web security academy!

this will be an eternally evolving space as i progress through them over time, aiming to complete them at some point soon (2026-willing).

## completion tree:

### client-side vulnerabilities:
- [ ] JWTs
- [ ] [XSS](/writeups/burp/client-side/XSS/xss)
	- [x] [Reflected XSS](/writeups/burp/client-side/xss/reflected-xss)
	- [x] [Stored XSS](/writeups/burp/client-side/xss/stored-xss)
	- [x] [Resources & Cheat sheet](/writeups/burp/client-side/xss/cheat-sheet)
	- [x] CSP
	- [x] Preventing XSS
	- [x] [XSS Contexts & Exploits](/writeups/burp/client-side/xss/xss-contexts/xxs/contexts)
		- [ ] Client-side template injection XSS
	- [x] [DOM-based XSS](/writeups/burp/client-side/xss/dom-xss/dom-xss)
		- [x] [Exploiting DOM XSS](/writeups/burp/client-side/xss/dom-xss/exploiting-dom-xss)
- [ ] DOM-based vulnerabilities
- [x] Websockets
- [x] CSRF
	- [x] Bypass SameSite Cookies
	- [x] Bypassing CSRF Tokens
	- [x] Bypassing Referer-based CSRF Defences
- [x] CORS
### server-side vulnerabilities:
- [ ] [Access Control](/writeups/burp/server-side/access-control)
- [ ] [API Testing](/writeups/burp/server-side/api-testing)
	- [ ] [API Auth Bypass Testing Methods](/writeups/burp/server-side/api-testing/api-auth-bypass-testing-methods.html)
	- [ ] [API Basics](/writeups/burp/server-side/api-testing/api-basics)
		- [ ] [Server-side Parameter Pollution](/writeups/burp/server-side/api-testing/server-side-parameter-pollution)
			- [ ] [REST API Paths](/writeups/burp/server-side/api-testing/sspp/rest-api-paths)
			- [ ] [Structured Data](/writeups/burp/server-side/api-testing/sspp/structured-data)
- [ ] Authentication
	- [ ] [Alternative Authentication forms](/writeups/burp/server-side/authentication/alternative-authentication-forms)
	- [ ] [Authentication](/writeups/burp/server-side/authentication/authentication)
	- [ ] [MFA & 2FA Authentication](/writeups/burp/server-side/authentication/mfa-and-2fa-authentication)
	- [ ] [Password-based Authentication](/writeups/burp/server-side/authentication/password-based-authentication)
- [ ] [Base HTML Tag Injection](/writeups/burp/server-side/base-html-tag-injection)
- [ ] [Business Logic Vulnerabilities](/writeups/burp/server-side/business-logic-vulnerabilities)
- [ ] [File Upload Vulnerabilities](/writeups/burp/server-side/file-upload-vulnerabilities)
- [ ] [File Extension Bypass List](/writeups/burp/server-side/file-upload-vulnerabilities/file-extension-bypass-list)
- [ ] [File Upload Vulns](/writeups/burp/server-side/file-upload-vulnerabilities/file-upload-vulns)
- [ ] [XSS via SVG upload](/writeups/burp/server-side/file-upload-vulnerabilities/xss-via-svg-upload)
- [ ] [HTTP Method Override Checks](/writeups/burp/server-side/http-method-override-checks)
- [ ] [OS Command or Shell Injection](/writeups/burp/server-side/os-command-or-shell-injection)
- [ ] [Path Traversal](/writeups/burp/server-side/path-traversal)
- [ ] [SQLi](/writeups/burp/server-side/sqli)
	- [ ] [Blind SQLi](/writeups/burp/server-side/sqli/blind-sqli)
	- [ ] [SQLi](/writeups/burp/server-side/sqli/sqli)  
	- [ ] [UNION Attacks](/writeups/burp/server-side/sqli/union-attacks)  
- [ ] [SSRF (Server Side Request Forgery)](/writeups/burp/server-side/ssrf-server-side-request-forgery)  
- [ ] [Testing MFA](/writeups/burp/server-side/testing-mfa)  
- [ ] [XXE Injection](/writeups/burp/server-side/xxe-injection)
	- [ ] [Blind XXE](/writeups/burp/server-side/xxe-injection/blind-xxe)
	- [ ] [XXE Injection](/writeups/burp/server-side/xxe-injection/xxe-injection)
### advanced topics: