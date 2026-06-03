### Server-side parameter pollution:
*(NOT [server-side prototype pollution](https://portswigger.net/web-security/prototype-pollution/server-side).)*
**`Server-side parameter pollution`**: when a site **embeds user input** in a **server-side request to an internal API** without adequate encoding/sanitisation, allowing an attacker to **inject/manipulate parameters.**
Can occur within:
- request query (`?=`) parameters
- form fields
- headers
- URL path parameters
- ... and more

### Testing Methodology:
- Place **URL-encoded query syntax characters** (encoded forms of `#, &, =`) in your input and observe how the application responds.

**Example:**
- For an application that searches for users, the browser makes a request: `GET /userSearch?name=peter&back=/home`.
- These parameters may be passed to a back-end API call: 
  `GET /users/search?`**`name=peter`**`&publicProfile=true`
### Testing cases:
#### Truncating query strings (`#`):
**Truncate query strings** with `#`, but URL-encoded (`%23`) 
- *Must be URL-encoded as otherwise, front-end app parses it as fragment identifier & isn't passed to backend API*

E.g. for:
1. **Browser:** `GET /userSearch?name=peter`**`%23foo`**`&back=/home`
2. **API:** `GET /users/search?name=peter`**`#foo`**`&publicProfile=true`
- If the response returns the user `peter`, the server-side query may have been **truncated** (removing requirement for `&publicProfile=true`). 
- If an `Invalid name` error message is returned, the application may have treated `foo` as part of the username. This suggests that the server-side request may **not have been truncated**.

#### Injecting valid/invalid parameters (`&`):
#hidden-api-parameters
Using any parameters found, test responses when injecting **invalid/valid parameters**. 

1. **Browser:** `GET /userSearch?name=peter`**`%26name=carlos`**`&back=/home`
2. **API:** `GET /users/search?name=peter`**`&name=carlos`**`&publicProfile=true`

Could use this to:
- **Override original parameter**, depending on **how the API interprets two duplicate parameters**.

To do so, for forms where parameters are passed in body/in URL, add **additional URL-encoded parameters** behind them and **observe error message responses.**

### Example:
**Find additional fields used in request by truncating the original request with a `#` (`%23`)**. Suggests that the request requires a `field=X` parameter to pass to a backend API.
![](attachments/Screenshot%202026-03-10%20at%209.48.05%20am.png)

Can verify this by **brute-forcing potential field names with Intruder**, and testing any hits to see if the API parses & returns info about them.
![](attachments/Screenshot%202026-03-10%20at%209.49.43%20am.png)

**Using a field found in a static `.js` file, picked up by [JS Link Finder](https://portswigger.net/bappstore/0e61c786db0c4ac787a08c4516d52ccf)**
![](attachments/Screenshot%202026-03-10%20at%209.46.03%20am.png)
![](attachments/Screenshot%202026-03-10%20at%209.46.21%20am.png)
... to find & pass the `reset_token` parameter to the internal API to reveal the one-time-use token to reset the password.
![](attachments/Screenshot%202026-03-10%20at%209.47.25%20am.png)
