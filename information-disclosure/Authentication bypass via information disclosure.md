## Authentication bypass via information disclosure

**Lab:** Client-side desync / access control bypass using `X-Custom-IP-Authorization`

App restricts `/admin` to requests appearing to come from `localhost`, using a custom header to track "trusted" IPs. Using the `TRACE` HTTP method reveals that header is being auto-injected by an intermediary — letting an attacker spoof it directly.

### The Vulnerability

```
GET /admin        ✗ "only accessible from localhost or as admin"
TRACE /admin       → response echoes back the exact request Burp/proxy sent,
                      revealing: X-Custom-IP-Authorization: <your IP>
```

The `TRACE` method makes the server echo the raw request back verbatim — exposing a header (`X-Custom-IP-Authorization`) that a front-end proxy silently adds to every request, containing the client's IP. The app uses this header (not real network-level IP checking) to decide if a request "came from localhost."

### The Attack

1. Send `GET /admin` in Repeater → blocked, error mentions localhost/admin-only access
2. Send `TRACE /admin` → response reflects the full request, revealing the auto-added `X-Custom-IP-Authorization` header
3. Go to **Proxy > Match and replace**, add a new rule:
   - Match: (empty)
   - Type: Request header
   - Replace: `X-Custom-IP-Authorization: 127.0.0.1`
4. Test the rule → confirms Burp appends/overwrites the header on every request
5. Browse to the home page / `/admin` again → header now spoofs `127.0.0.1` → access granted
6. Use the now-accessible admin panel to delete `carlos`

### Why It's Vulnerable

- Access control decision is based on a client-controllable/spoofable HTTP header, not real network-level origin
- `TRACE` method left enabled, letting the client see exactly how the server (or a proxy in front of it) modifies/adds headers to requests
- Trust boundary confusion: the app assumes a header set by an internal proxy can never be set by an external client, but nothing stops the client from just sending that header directly

### Prevention

- Never use client-supplied or header-based values as the sole basis for trust/access decisions
- If an internal proxy adds trust headers, strip/overwrite any client-supplied version of that header at the network edge before it reaches the app
- Disable the `TRACE` HTTP method on production servers (also historically relevant to XST - cross-site tracing attacks)
- Enforce real IP-based access control at the network/firewall layer, not via forgeable application headers
- Apply proper authentication/authorization for admin functionality regardless of "trusted network" assumptions

### What I Learnt

- `TRACE` isn't just a theoretical/legacy method — it can be a direct information-disclosure tool, since it reflects the exact request as received, exposing headers added by intermediaries the client normally never sees.
- Header-based trust (e.g., "if this header says localhost, treat it as localhost") is trivially bypassable since headers are just text the client can set — there's no cryptographic or network guarantee behind them.
- Burp's **Match and Replace** is useful for persistently injecting/overwriting a header across all outgoing requests, not just one-off Repeater tests — good for exploiting this class of bug across a full browsing session.
- Always check for legacy/uncommon HTTP methods (`TRACE`, `OPTIONS`, `PUT`, `DELETE`) being enabled — they can leak info or expose extra functionality the app didn't intend to expose.
