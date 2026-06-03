---
sticker: emoji//2705
---
**Stored XSS:** when an application receives data from untrusted sources, & unsafely includes that data within later HTTP responses. It enables attacks that are self-contained within the application itself, as no external delivery mechanism is required to target users.

### Testing for Stored XSS
- Can be time consuming as all "entry points" must be tested:
	- **Parameters or data in URL query string/message body**.
	- The **URL file path**.
	- **HTTP request headers** (even ones that might not be exploitable in relation to reflected XSS).
	- **Out-of-band data delivery routes** - e.g. emails, third party messages (e.g. twitter), other sites (e.g. news aggregator).
- Systematically submit unique data values to each entry point, & monitor responses to detect when/where values appear & whether they are **stored** or just **reflected.**
	- **Consider "hidden" exit points** like "recent searches" or obscure audit logs visible to only some users.
- **Determine** the [XSS Context](XSS%20Contexts/XSS%20Contexts.md) & adjust payloads accordingly.