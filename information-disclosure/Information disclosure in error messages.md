# Information disclosure in error messages

App throws unhandled exceptions on malformed input and returns a full stack trace, leaking internal implementation details — including the exact third-party framework version in use.

### The Vulnerability

```
GET /product?productId=1            ✓ 200 (normal response)
GET /product?productId="example"    ✗ 500 (stack trace leaked)
```

Sending a non-integer value for `productId` triggers a type-conversion exception. Instead of a generic error page, the server returns the raw stack trace — revealing it's running **Apache Struts 2 2.3.31**.

### The Attack

1. Browse a product page, capture the request in Burp Proxy → HTTP history
2. Find `GET /product?productId=1`, send to Repeater
3. Change `productId` to a non-integer value: `productId="example"`
4. Send the request → exception thrown, full stack trace returned in response
5. Read the stack trace to identify the framework name + version
6. Submit `2 2.3.31` as the lab solution

### Why It's Vulnerable

- App doesn't validate/sanitize input type before using it
- Unhandled exceptions aren't caught — default verbose error handling is left enabled
- Stack traces expose internal class names, file paths, and library/framework versions
- Attackers use this info for recon — matching a disclosed version against known public CVEs (e.g. Struts OGNL injection RCEs) to plan further attacks

### Prevention

- Validate and sanitize all user input before processing (type checks, whitelisted formats)
- Disable verbose/debug error pages in production — return generic error messages to users
- Log full stack traces server-side only, never in the HTTP response
- Keep third-party frameworks patched/updated regardless — don't rely on obscurity even if errors are hidden
- Use a global exception handler that catches unhandled exceptions and returns a safe, generic response

### What I Learnt

- Type-confusion input (e.g. passing a string where an integer is expected) is a simple, low-effort way to probe for verbose error handling.
- Stack traces are a common but often overlooked info-disclosure vector — they can hand attackers exact framework/library versions for free.
- Information disclosure isn't exploitation by itself, but it's often the first recon step toward finding and using a real, version-specific exploit.
