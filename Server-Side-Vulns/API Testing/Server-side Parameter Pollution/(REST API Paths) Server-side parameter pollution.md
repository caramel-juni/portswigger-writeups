#REST-API may place parameter names and values in the URL path, rather than the query string. e.g.:

`/api/users/123`
- `api` = root endpoint
- `users` = resource
- `123` = a parameter (like a userID)

**E.g.** for an application where you edit user profiles, requests sent to the following endpoint:
`GET /edit_profile.php?name=peter`
...Might result in the following server-side API request:
`GET /api/private/users/peter`

#### Path Traversal:
May be able to **manipulate** this user-injected value with URL-encoded **path traversal characters** (`/../`, i.e. `%2f..%2f`) to break into other endpoints, or traverse the **internal API**. 
**E.g.** 
- `GET /edit_profile.php?name=peter%2f..%2fadmin` 
- `GET /api/private/users/peter/../admin`
- `GET ../../../../api/internal/v1/users/admin/field/passwordResetToken%23`

## Example:

#### Discovering the API:
Notice that the `/forgot-password` request accepts/uses query parameters the response body for `POST` requests, suggesting it's potentially supplying them to a back-end API endpoint. 
#### Determining whether input is placed in the URL path of server-side API request
**Send a variety of requests** with a modified `username` parameter value to determine whether the input is placed in the URL path of a server-side request without escaping.
- `administrator%23` (`#`)--> `Invalid Route`. *Suggests server may have placed the input in the path of a server-side request, and that the fragment has truncated some trailing data*
  e.g. `GET /api/internal/v1/users/forgot-password`
- `administrator%3f` (`?`)--> `Invalid Route`. *Suggests that the input may be placed in a URL path, as `?` denotes start of query string.*
  e.g. 
- `./administrator` --> `Same Response`. *Suggests the request may have **accessed the same URL path**, indicates the input may be placed in the URL path.*
- `../administrator` --> `Invalid route`. Suggests the request may have escaped the original path, & accessed an invalid URL path.
#### Finding the API spec:
Via path traversal & query truncation with URL-encoded `#`, can attempt to find API docs with a wordlist of common documentation endpoints/locations.
- e.g. `&username=../../../../../../openapi.json%23`
![](attachments/Screenshot%202026-03-10%20at%203.53.23%20pm.png)
Indicates the internal API endpoint being interacted with is: `/api/internal/v1/users/{username}/field/{field}`

#### Access & test `field` parameters
- `administrator/field/email%23`
![](attachments/Screenshot%202026-03-10%20at%204.04.45%20pm.png)
- `administrator/field/passwordResetToken%23`
![](attachments/Screenshot%202026-03-10%20at%204.42.32%20pm.png)
As this API path only accepts the `email` parameter, we can combine the previous path traversal to target a different API path - notably, the `passwordResetToken` field, found in the **static `.js` file, picked up by [JS Link Finder](https://portswigger.net/bappstore/0e61c786db0c4ac787a08c4516d52ccf)**:
![](attachments/Screenshot%202026-03-10%20at%209.46.03%20am.png)
![](attachments/Screenshot%202026-03-10%20at%203.04.31%20pm.png)
To do this, we need to:
1. Break out of the reduced functionality API path
2. Access the full internal API path, at the `passwordResetToken` field.
We do this by sending: `username=../../../../api/internal/v1/users/administrator/field/passwordResetToken%23` (using `%23` to truncate the query)
 ![](attachments/Screenshot%202026-03-10%20at%204.48.32%20pm.png)
Can then reset the password by supplying said token to the `/forgot-password` URL within the query string `?passwordResetToken=XXXX`
**Using a field found in a
