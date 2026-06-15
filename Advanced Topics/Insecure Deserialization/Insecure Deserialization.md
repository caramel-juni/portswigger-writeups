---
sticker: emoji//2705
---
## What is serialization?
The process of **converting complex data structures** (e.g. objects & their properties) into a **flatter format**, to be **transferred as a series of bytes**. The state of the object & its attributes are preserved, allowing for it to be **sent over a network/API call/between application components**, **processed in memory**, written to a **file** or a **database**.
 - **ALL attributes** are serialised **apart from any private fields, if marked as `transient`**.
 - **Serialisation formats vary** depending on language - can be **binary** (Java), **strings** (PHP), etc.
 - May be **referred to with different terms**, depending on language: #marshalling (Ruby) or #pickling (Python).
##### Serialization example:
- `$user->name = "carlos"; $user->isLoggedIn = true;` --> **`O:4:"User":2:{s:4:"name":s:6:"carlos";s:10:"isLoggedIn":b:1;}`** (**serialized** data, `PHP`)

### Deserialization:
The process of **restoring a stream of serialized bytes** back into its **original form** (e.g. an object & its properties), to be interacted with & processed.
##### Deserialization example:
- `O:4:"User":2:{s:4:"name":s:6:"carlos";s:10:"isLoggedIn":b:1;}` --> - **`$user->name = "carlos"; $user->isLoggedIn = true;`** (**DEserialized** data, `PHP`)
![](attachments/Insecure%20Deserialization.png)

## Insecure deserialization / "Object injection"
When **user-controllable data is deserialized** by a site, allowing adversaries to **manipulate serialized objects** and/or **inject entirely different objects/classes** to be processed by the application's code. As serialized data will be **deserialized & directly instantiated on the server**, often regardless of the expected object class, it enables **damage to be done EVEN IF any exceptions are thrown** or the website **doesn't interact with the object**.

##### Impact: 
Can be extremely severe, as provides **entry point & expanded attack surface** to **reuse application code**, **objects & methods** in unintended ways, e.g. to achieve:
- *Privilege escalation*
- *RCE*
- *Arbitrary file access*
- *DoS*

#### User input should almost NEVER be deserialized, because:
- **Validation & sanitization is flawed**, as it:
	- **Cannot account for every eventuality**
	- Relies on **checking data after being deserialized**, where the **damage could've already been done**
- Using **binary formats is not obfuscation** - can be as easy to manipulate as string-based serialization, with tooling
- Websites **implement countless libraries**, creating a **massive pool of classes and methods** that can be potentially instantiated, and **chained together** to access/manipulate data.

### Preventing insecure deserialization issues:
- **Never deserialize untrusted/user input**
- *If* absolutely necessary, use robust verification measures performed **BEFORE the deserialization process** - e.g. digital message signatures
- Implement **class-specific serialization methods**, to control the values of expected fields (& prevent all properties of an object being accessed)

