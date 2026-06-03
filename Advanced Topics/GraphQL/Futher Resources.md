## Bypassing `GET` Mutation Restrictions (e.g. CSRF + SSRF)
- [Awesome GraphQL CSRF and SSRF](https://lachlan.nz/blog/graphql-csrf-and-ssrf/)
**Exploiting:**
- Poor whitelisting: **beginning request with null byte/comment** to bypass logic like `if stripped_query starts with "mutation", throw NOT ALLOWED`
- Is case-jumbling blocked? e.g. `mUtaTiOn`
- What if you put both a `query` and a `mutation` in the **same request?**
- Putting a `query?` inside a `POST` request: e.g. `POST /api/graphql?query=mutation+%7B+doSomeBadThings()+%7D HTTP/1.1`
*Can be very dangerous as **GraphQL APIs can sequentially execute a large number of operations in a single request - batch queries, multiple changes with one CSRF.***
### custom graphQL-filter columns in burp
``` js
##Burp Custom Columns: 
String q = requestResponse.request().parameterValue("query", HttpParameterType.JSON);
if (q != null) {
   return q.split("\\{|\\(")[0];
}
return "";
```

### Labs:
- https://www.hackerone.com/blog/graphql-week-hacker101-capture-flag-challenges

### Books:
- [The Black Hat GraphQL Book Repository (Files/labs from book) - GitHub](https://github.com/dolevf/Black-Hat-GraphQL) + Actual Book

### Wordlists:
- https://github.com/Escape-Technologies/graphql-wordlist?

### Plugins/Apps:
- inQL
- [Graphquail](https://github.com/forcesunseen/graphquail) (passive detection and building of a GraphQL schema from proxy traffic)
- [graphw00f - GraphQL Server Fingerprinting](https://github.com/dolevf/graphw00f)
- [Altair](https://altairgraphql.dev/) - connects to remote GraphQL servers and returns results similarly to the way GraphiQL Explorer would
- **CrackQL** - tool to better optimize brute-force attacks against auth'd API actions
- **GraphQL Cop** - dedicated GraphQL security auditing utility
- **graphql-path-enum**
- **Commix** - find and *exploit command injection vulnerabilities* in an automated fashion by fuzzing various parts of an HTTP request, such as query parameters or the request body, using specialized payloads
- **BatchQL** - identify batching-based vulns - DoS, CSRF, and information disclosure


### videos:
- all of [insiderPHD's GraphQL API hacking videos](https://www.youtube.com/playlist?list=PLbyncTkpno5HqX1h2MnV6Qt4wvTb8Mpol)
- [So you want to Hack GraphQL APIs ??](https://www.youtube.com/watch?v=OOztEJu0Vts)
## Misc.
- [https://github.com/dolevf/graphql-cop](https://github.com/dolevf/graphql-cop): Test common misconfigurations of graphql endpoints
- [https://github.com/assetnote/batchql](https://github.com/assetnote/batchql): GraphQL security auditing script with a focus on performing batch GraphQL queries and mutations.
- [https://github.com/dolevf/graphw00f](https://github.com/dolevf/graphw00f): Fingerprint the graphql being used
- [https://github.com/gsmith257-cyber/GraphCrawler](https://github.com/gsmith257-cyber/GraphCrawler): Toolkit that can be used to grab schemas and search for sensitive data, test authorization, brute force schemas, and find paths to a given type.
- [https://blog.doyensec.com/2020/03/26/graphql-scanner.html](https://blog.doyensec.com/2020/03/26/graphql-scanner.html): Can be used as standalone or [Burp extension](https://github.com/doyensec/inql).