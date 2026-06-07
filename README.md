# portswigger writeups
here lie all my notes and lab writeups for portswigger's wonderful web security academy!

this will be an eternally evolving space as i progress through them over time, aiming to complete them at some point soon (2026-willing).

---

## 🌳 completion tree of topics ~ 🌳


### Client-Side-Vulns

* [x] [CORS](./Client-Side-Vulns/CORS.md)
* [x] [DOM based Vulnerabilities](./Client-Side-Vulns/DOM%20based%20Vulnrabilities.md)
* [ ] [JWTs](./Client-Side-Vulns/JWTs.md)
* [x] [WebSockets](./Client-Side-Vulns/WebSockets.md)

---

#### CSRF

* [x] [CSRF](./Client-Side-Vulns/CSRF/CSRF.md)
* [x] [Bypassing CSRF Tokens](./Client-Side-Vulns/CSRF/Bypassing%20CSRF%20Tokens.md)
* [x] [Bypassing Referer-based CSRF Defences](./Client-Side-Vulns/CSRF/Bypassing%20Referer-based%20CSRF%20Defences.md)
* [x] [Bypass SameSite Cookie Restrictions](./Client-Side-Vulns/CSRF/Bypass%20SameSite%20cookie%20restrictions.md)

---

#### XSS

* [x] [XSS](./Client-Side-Vulns/XSS/XSS.md)
* [x] [CSP](./Client-Side-Vulns/XSS/CSP.md)
* [ ] [Preventing XSS](./Client-Side-Vulns/XSS/Preventing%20XSS.md)
* [x] [Reflected XSS](./Client-Side-Vulns/XSS/Reflected%20XSS.md)
* [x] [Stored XSS](./Client-Side-Vulns/XSS/Stored%20XSS.md)
* [x] [Resources & Cheat Sheets](./Client-Side-Vulns/XSS/Resources%20%26%20Cheat%20sheets.md)

##### DOM-based XSS

* [x] [DOM-based XSS](./Client-Side-Vulns/XSS/DOM-based%20XSS/DOM-based%20XSS.md)
* [ ] [Exploiting DOM XSS](./Client-Side-Vulns/XSS/DOM-based%20XSS/Exploiting%20DOM%20XSS.md)

##### XSS Contexts

* [x] [XSS Contexts](./Client-Side-Vulns/XSS/XSS%20Contexts/XSS%20Contexts.md)
* [ ] [Client-side Template Injection XSS](./Client-Side-Vulns/XSS/XSS%20Contexts/Client-side%20template%20injection%20XSS.md)

---
---


## Server-Side-Vulns

* [x] [Base HTML Tag Injection](./Server-Side-Vulns/Base%20HTML%20Tag%20Injection.md)
* [x] [Business Logic Vulnerabilities](./Server-Side-Vulns/Business%20Logic%20Vulnerabilities.md)
* [x] [HTTP Method Override Checks](./Server-Side-Vulns/HTTP%20Method%20Override%20Checks.md)
* [x] [OS Command or Shell Injection](./Server-Side-Vulns/OS%20Command%20or%20Shell%20Injection.md)
* [x] [Path Traversal](./Server-Side-Vulns/Path%20Traversal.md)
* [x] [SSRF (Server Side Request Forgery)](./Server-Side-Vulns/SSRF%20%28Server%20Side%20Request%20Forgery%29.md)
* [x] [Testing MFA](./Server-Side-Vulns/Testing%20MFA.md)
* [x] [Information disclosure vulnerabilities](./Server-Side-Vulns/Information%20disclosure%20vulnerabilities.md)

---

#### Access Control

* [ ] [Access Control](./Server-Side-Vulns/Access%20Control/Access%20Control.md)

---

### API Testing

* [x] [API Basics](./Server-Side-Vulns/API%20Testing/API%20Basics.md)
* [x] [API Auth Bypass Testing Methods](./Server-Side-Vulns/API%20Testing/API%20Auth%20Bypass%20Testing%20Methods.html)

#### Server-side Parameter Pollution

* [x] [Server-side Parameter Pollution](./Server-Side-Vulns/API%20Testing/Server-side%20Parameter%20Pollution/Server-side%20Parameter%20Pollution.md)
* [x] [(API) Server-side Parameter Pollution](./Server-Side-Vulns/API%20Testing/Server-side%20Parameter%20Pollution/%28API%29%20Server-side%20parameter%20pollution%20.md)
* [x] [(REST API Paths) Server-side Parameter Pollution](./Server-Side-Vulns/API%20Testing/Server-side%20Parameter%20Pollution/%28REST%20API%20Paths%29%20Server-side%20parameter%20pollution.md)
* [x] [(Structured Data) Server-side Parameter Pollution](./Server-Side-Vulns/API%20Testing/Server-side%20Parameter%20Pollution/%28Structured%20Data%29%20Server-side%20parameter%20pollution.md)

---

#### Authentication

* [x] [Alternative Authentication Forms](./Server-Side-Vulns/Authentication/Alternative%20Authentication%20forms.md)
* [x] [MFA & 2FA Authentication](./Server-Side-Vulns/Authentication/MFA%20%26%202FA%20Authentication.md)
* [x] [Password-based Authentication](./Server-Side-Vulns/Authentication/Password-based%20Authentication.md)

---

#### File Upload Vulnerabilities

* [x] [File Upload Vulns](./Server-Side-Vulns/File%20Upload%20Vulnerabilities/File%20Upload%20Vulns.md)
* [x] [File Extension Bypass List](./Server-Side-Vulns/File%20Upload%20Vulnerabilities/File%20Extension%20Bypass%20List.md)
* [x] [XSS via SVG Upload](./Server-Side-Vulns/File%20Upload%20Vulnerabilities/XSS%20via%20SVG%20upload%20.md)

---

#### SQLi

* [x] [SQLi](./Server-Side-Vulns/SQLi/SQLi.md)
* [x] [Blind SQLi](./Server-Side-Vulns/SQLi/Blind%20SQLi.md)
* [x] [UNION Attacks](./Server-Side-Vulns/SQLi/UNION%20Attacks.md)

---

#### XXE Injection

* [x] [XXE Injection](./Server-Side-Vulns/XXE%20Injection/XXE%20Injection.md)
* [x] [Blind XXE](./Server-Side-Vulns/XXE%20Injection/Blind%20XXE.md)

---
---

### "Advanced" Topics

#### Authentication Flows
##### OAuth

* [x] [OAuth](./Advanced%20Topics/Authentication%20Flows/OAuth/OAuth.md)
* [x] [OAuth Flow](./Advanced%20Topics/Authentication%20Flows/OAuth/OAuth%20Flow.md)
* [x] [OAuth Attacks](./Advanced%20Topics/Authentication%20Flows/OAuth/OAuth%20Attacks.md)

##### OIDC

* [ ] [OIDC](./Advanced%20Topics/Authentication%20Flows/OIDC.md)

##### SAML

* [x] [SAML Explained](./Advanced%20Topics/Authentication%20Flows/SAML/SAML%20Explained.md)
* [x] [SAML Attacks](./Advanced%20Topics/Authentication%20Flows/SAML/SAML%20Attacks.md)

---

#### GraphQL

* [x] [GraphQL](./Advanced%20Topics/GraphQL/GraphQL.md)
* [x] [Exploiting GraphQL](./Advanced%20Topics/GraphQL/Exploiting%20GraphQL.md)
* [x] [Further Resources](./Advanced%20Topics/GraphQL/Futher%20Resources.md)

---

#### HTTP Host Header

* [x] [HTTP Host Header](./Advanced%20Topics/HTTP%20Host%20Header/HTTP%20Host%20Header.md)
* [x] [Finding HTTP Host Header Vulnerabilities](./Advanced%20Topics/HTTP%20Host%20Header/Finding%20HTTP%20Host%20Header%20Vulns.md)
* [x] [Exploiting HTTP Host Header Vulnerabilities](./Advanced%20Topics/HTTP%20Host%20Header/Exploiting%20HTTP%20Host%20Header%20Vulns.md)

---

#### HTTP Request Smuggling

* [ ] [HTTP Request Smuggling](./Advanced%20Topics/HTTP%20Request%20Smuggling/HTTP%20Request%20Smuggling.md)
* [ ] [HTTP Request Smuggling Types](./Advanced%20Topics/HTTP%20Request%20Smuggling/HTTP%20Request%20Smuggling%20Types.md)
* [ ] [Using HTTP Request Smuggler Extension](./Advanced%20Topics/HTTP%20Request%20Smuggling/Using%20HTTP%20Request%20Smuggler%20Extension.md)

---

#### Insecure Deserialization

* [ ] [Insecure Deserialization](./Advanced%20Topics/Insecure%20Deserialization/Insecure%20Deserialization.md)
* [ ] [Identifying insecure deserialization](./Advanced%20Topics/Insecure%20Deserialization/Identifying%20insecure%20deserialization.md)
* [ ] [Exploiting insecure deserialization](./Advanced%20Topics/Insecure%20Deserialization/Exploiting%20insecure%20deserialization.md)

---

#### Server Side Template Injection

* [x] [Server Side Template Injection](./Advanced%20Topics/Server%20Side%20Template%20Injection/Server%20Side%20Template%20Injection.md)
* [x] [Exploiting SSTI](./Advanced%20Topics/Server%20Side%20Template%20Injection/Exploiting%20SSTI.md)

---

#### Web Cache Poisoning

* [x] [Web Cache Poisoning](./Advanced%20Topics/Web%20Cache%20Poisoning/Web%20Cache%20Poisoning.md)
* [x] [Exploiting Cache Design Flaws](./Advanced%20Topics/Web%20Cache%20Poisoning/Exploiting%20cache%20design%20flaws.md)
* [ ] [Exploiting Cache Implementation Flaws](./Advanced%20Topics/Web%20Cache%20Poisoning/Exploiting%20cache%20implementation%20flaws.md)

---
---


## progress

this repo grows whenever i complete a lab, discover something interesting, or forget the same payload for the bajillionth time.
there may be (quite a few) obsidian-based artifacts left over, as this is a copy of my notes in obsidian. i am yet to fix (a lot) of image links.
eventually i'd like to add:

* lab references for every topic
* methodology checklists
* payload collections
* quick-reference cheat sheets
* cross-links between related vulnerabilities

---

## disclaimer

all notes are for educational purposes and are based primarily on portswigger academy content, supplemented by personal notes and lab writeups.

_only test systems you own or have explicit permission to assess pls_
