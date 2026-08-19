## SSRF - Basic Access to Admin Interface

**Lab:** Basic SSRF against the local server

**Difficulty:** Apprentice

Application has a stock check feature that fetches data server-side from an internal system, with no restriction on the target URL.

### The Vulnerability

**Direct access (blocked):**

```
GET /admin
```
✗ Not directly accessible — presumably restricted to internal/localhost requests only

**Via stock check (works):**

```
POST /product/stock
stockApi=http://localhost/admin
```
✓ Server-side request originates from localhost, bypassing the restriction

### Why This Works

- The admin panel trusts requests based on origin (localhost), not authentication
- The stock check feature fetches whatever URL is supplied in `stockApi` server-side
- No validation is performed on the URL — no scheme, host, or path checks
- The server becomes a proxy: it makes the request *for* the attacker, satisfying the localhost-only restriction

### The Exploit

1. Browse to `/admin` directly — confirm access is denied.
2. Visit any product page and click **Check stock**.
3. Intercept the request in Burp Suite and send it to Repeater.
4. Change the `stockApi` parameter to:

```
http://localhost/admin
```

5. The response body returns the admin interface HTML. Read it to find the delete-user endpoint:

```
http://localhost/admin/delete?username=carlos
```

6. Submit that URL as the `stockApi` value to trigger the delete server-side.

### Prevention

- Whitelist allowed hosts/URLs for server-side requests — don't accept arbitrary user-supplied URLs
- Enforce authentication and authorization on the admin interface itself, not just network origin
- Disable unused URL schemes and block requests to loopback/internal address ranges at the network layer
