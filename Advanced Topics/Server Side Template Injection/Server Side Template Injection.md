---
sticker: emoji//2705
---
### What is SSTI?
#ssti #not-sti
**Server Side Template Injection:** Where user input is **directly** passed into server-side templates, instead of being sanitised and treated as data. This allows adversaries to inject **payloads inside server-side template syntax** to execute commands server-side. 

Template engines generate content (for webpages) by combining fixed **server templates (how a page is constructed/built)** with **volatile (often user-generated) data**. Enables dynamic updates of pages, etc. SSTI can be extremely dangerous as it can result in a direct path to executing server-side code, exfiltration of arbitrary data or accessing and modifying server-side files.

### How do the vulnerabilities arise?
- When user input is **concatenated into templates instead of passed in as data.**

**Example of input passed in AS DATA** *(safe)*:
- `$output = $twig->render("Dear {first_name},", array("first_name" => $user.first_name) );`
Reads & appends properties of an object, as **data**, no additional functions/code sinks performed/executed.

**Example of input DIRECTLY CONCATENATED into template** *(DANGEROUS)*:
- `$output = $twig->render("Dear " . $_GET['name']);
Here, **part of the template itself** is being **dynamically generated** (via the `$_GET` parameter `name`). Thus, by placing **another template statement into `name`** (e.g. via `http://vulnerable-website.com/?name={{bad-stuff-here}}`), additional commands can be executed.

### How to detect SSTI?
- **Fuzz for templates using common SSTI template expressions**, such as `${{<%[%'"}}%\`
- Questioning & testing as many forms of input where user **input is used to influence/create dynamic page content** 
- Usually occurs in two contexts:
#### Plaintext SSTI Context
Most server side templating engines allow free input of content **via HTML tags** or the **server-side template's expression syntax**, which will be **rendered/executed on the backend** before being **returned as HTML.**

As such, can **often be mistaken for XSS**, but **always remember to attempt to test SSTI evaluations**, such as:
**Vulnerable code:** `render('Hello ' + username)`
**Injection test:** `http://vulnerable-website.com/?username={7 * 7}`
- If this returns `Hello 49`, know the expression has been **evaluated server-side** & you have SSTI.

#### Code SSTI Context
Where user input/**a user controlled variable name** is **placed directly within a template expression**. 
**Vulnerable Code:** 
`greeting = getQueryParameter('greeting')`
`engine.render("Hello {{"+greeting+"}}", data)`
**Vulnerable Input:** `http://vulnerable-website.com/?greeting=data.username`
**Output:** `Hello Carlos`
##### Testing method:
1. **Establish the parameter doesn't contain a direct XSS vulnerability** by injecting arbitrary HTML into the value (`http://vulnerable-website.com/?greeting=data.username<tag>)
2. **IF NO XSS** (blank output, encoded tags, or error message): attempt to **break out of the statement** using common #SST syntax ***and then inject HTML***: `http://vulnerable-website.com/?greeting=data.username}}<tag>`
	1. **IF ERROR/BLANK OUTPUT:** try a different SSTI language (fuzz), and if no dice, **SSTI is not possible.**
	2. **IF OUTPUT & `HTML` is RENDERED CORRECTLY**: You have SSTI

### How to identify SSTI Template Engine?
Most templating languages **use very similar syntax** chosen **not to clash with HTML characters** (think of a Hugo blog - written in a mix of `{{expressions}}` and raw/static `<HTML>`).

Often **submitting invalid syntax** is enough to disclose the SST engine, and occasionally the version. A decision tree with several payloads can help narrow this down:
- `${7*7}`
	- **Yes:** try `a{*comment*}b`, then check evaluation:
		- **Yes:** Smarty
		- **No:** try `${"z".join("ab")}`, then check evaluation:
			- **Yes:** Mako
			- **No:** Unknown
	- **No:** try `{{7*7}}`, then check evaluation:
		- **Yes:** try `{{7*'7'}}`, then check evaluation:
			- Jinja
			- Twig
			- Other
		- **No:** Not vulnerable
![](attachments/Screenshot%202026-06-02%20at%201.03.45%20pm.png)*Payloads can occasionally return more than one successful response (e.g. `{{7*'7'}}` returns `49` in Twig and `7777777` in Jinja2)*

### Preventing SSTI:
- **Use a logic-less template engine** (e.g. `Moustache`) to separate server side logic from presentation of data.
- **Do not allow users to modify/submit new templates**
- Only execute user code in **sandboxed environments** (inherently difficult & prone to bypass)


