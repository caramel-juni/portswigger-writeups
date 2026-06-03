#SAML #SSO #XML
#### Helpful Resources
- [How to Hunt Bugs in SAML; a Methodology](https://epi052.gitlab.io/notes-to-self/blog/2019-03-07-how-to-test-saml-a-methodology/)
- [SAML Basics](https://angelica.gitbook.io/hacktricks/pentesting-web/saml-attacks/saml-basics) (Hacktricks)
- [SAML Attacks](https://angelica.gitbook.io/hacktricks/pentesting-web/saml-attacks) (Hacktricks)

**`SAML` (Security Assertion Markup Language)** is an `XML`-based standard for implementing SSO between one or multiple **Service Provider(s)** (`SP`) and one **Identity Provider** (`IDP`), where users can **utilize a single set of credentials to access multiple applications.**


1. **`SAML Assertion`**: The `XML` message containing the user's information & attributes.
2. **`Identity Provider (IdP)`**: The **service performing the authentication and issuing the Assertion**, via anything from username/password to MFA.
3. **`Service Provider (SP)`**: The **web application that the user wants to access.** 

![](attachments/saml-overview.png)

## Process:
1. The `User-Agent` (browser) tries to access the resource.
2. SP generates a `SAML Request` & sends to the user
   #saml-request
   ![](attachments/SAML.png)
    - `AssertionConsumerServiceURL`: Identifies **where the IdP should send the SAML Response to AFTER authentication**
    - `Destination`: Address to which the request should be sent (`IdP`)
    - `ProtocolBinding`: Typically accompanies the `AssertionConsumerServiceURL` attribute; **defines the mechanism** for transmitting `SAML` messages.
    - `saml:Issuer:` Identifies the **entity that generated the request message**
3. `SP` redirects user (`302`) to the `IdP`, with the `SAML` request **encoded into the HTTP `Location` header.**
	`https://shibdemo-idp.test.edu/idp/profile/SAML2/Redirect/SSO?SAMLRequest=fZJdT4MwFI....3D&RelayState=ss%3Amem%3A....`
	- `RelayState` identifies to the `SP` **who initially asked for the resource** for when the `SAML` response comes back.
	- `SAMLRequest` is compressed & encoded using the [Deflate compression](https://en.wikipedia.org/wiki/DEFLATE) algorithm, then `base64` encoding the result.
4. User authenticates to the IDP, with their request containing the `SAMLRequest`.
5. Upon success, the **`IdP` creates a `SAML` response, containing the `SAML` Assertions required by the `SP`** the user is trying to access. Typically (at minimum):
	- **Indication that the Assertion is from the correct `IdP`**
	- A **NameID** attribute specifying who the user is
	- A **digital signature**
6. `IdP` redirects to the **`SP`’s Assertion Consumer Service (`ACS`)** - the URL on which the `SP` expects to receive the `SAML` assertions. The ACS validates the SAML Reqponse here.
7. User can access the initially requested resource!

### SAML Response Breakdown:
#saml-response
- **`ds:Signature`**: An [XML Signature](https://www.w3.org/TR/xmldsig-core1/#sec-KeyInfo) protecting the message's integrity & authenticates the assertion's issuer (`IdP`). May contain two of these fields (one the message signature, and one the Assertion's signature.)
- **`saml:Assertion`**: Contains **information about the user’s identity** and potentially other user attributes.
- **`saml:Subject`**: Specifies the principal that is the subject of all of the statements in the assertion.
- **`saml:StatusCode`**: A code representing the **status of the activity** carried out in response to the corresponding request.
- **`saml:Conditions`**: Specifies things like the time an Assertion is valid and that the Assertion is addressed to a particular Service Provider.
- **`saml:AuthnStatement`**: States that the `IdP` authenticated the **Subject of the Assertion**.
- **`saml:AttributeStatement`**: Describes the **Subject of the Assertion.**
**Example:**
![](attachments/SAML-1.png)

### XML Signatures (& message signing)
#xml-signature 
Can be used to sign a **whole `XML` tree** or just **specific elements** (any `<XXXXXXX>` pair). Each resource to be signed **has its own `<Reference>` element** - denoting **which resource is signed.**
![](attachments/SAML-2.png)
#### XML Signature Types:
#enveloped-signatures #enveloping-signatures #detached-signatures
- **`Enveloped`:** when **the signature is a descendent of/inside the resource its signing** (this type is specified within the `<ds:Transform>` element) ![](attachments/SAML-Enveloped.png)
- **`Enveloping`:** when the **signature wraps around the resource in question.** ![](attachments/SAML-enveloping.png)
- **`Detached`:** when the **signature is separate from the resource to be signed**. ![](attachments/SAML-detached.png)

### *Now, see [SAML Attacks](SAML%20Attacks.md)*
