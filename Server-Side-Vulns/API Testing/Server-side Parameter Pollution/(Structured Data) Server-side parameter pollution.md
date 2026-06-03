## Testing for server-side parameter pollution in structured data formats

May be able to manipulate parameters to exploit vulnerabilities in the server's **processing of structured data formats** (e.g. `JSON` or `XML`).

If the **user input is added to the server-side structured data without adequate validation** or sanitization, may be able to **add/override/pollute parameters** and change unintended user attributes.

### Example:
Consider an application that enables users to edit their profile, then applies changes by requesting a server-side API. 
Could attempt to **break out of `JSON` parameter & add additional ones**, like so:
**User Request:**
``` HTTP
POST /myaccount
name=peter","access_level":"administrator
```
or
``` HTTP
POST /myaccount
{"name": "peter\",\"access_level\":\"administrator"}
```

**Backend API Request:**
``` HTTP
PATCH /users/7312/update
{name="peter","access_level":"administrator"}
```

