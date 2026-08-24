## SSRF - Bypassing Blacklist-Based Input Filters

**Lab:** SSRF with blacklist-based input filter

Filter blocks known-bad strings (`127.0.0.1`, `localhost`, `admin`) instead of properly validating URLs. Filter and backend HTTP client parse URLs differently — creates bypass gaps.

### The Vulnerability

```
stockApi=http://127.0.0.1/admin       ✗ blocked (host blacklisted)
stockApi=http://127.1/admin           ✗ blocked (path blacklisted)
stockApi=http://127.1/a%2564min       ✓ 200 (found!)
```

### The Attack

1. Intercept stock request, send to Repeater
2. Swap host to `127.1` (alt IPv4 shorthand) → bypasses host blacklist
3. Add `/admin` → blocked again (separate path blacklist)
4. Double-encode one letter in "admin" (`d` → `%64` → `%2564`) → filter's single decode never sees "admin", backend's decode reconstructs it
5. Combine: `http://127.1/a%2564min` → reach admin panel, delete target user

### Why It's Vulnerable

- Filter does raw string matching, not structural validation
- Filter and backend use different, inconsistent URL/IP parsers
- Only one decode pass before filter check, but request-maker decodes again later

### Prevention

- Use an allowlist, not a blacklist
- Validate after full decode/canonicalization — check resolved IP, not raw string
- Block private/loopback IP ranges structurally
- Restrict/disable redirect-following
- Network segmentation as backup

### What I Learnt

- IPv4 has legacy shorthand: fewer than 4 octets get combined into the last segment as one integer; decimal/octal forms also exist. Not all parsers support these — legacy `inet_aton()` leniency.
- Bypassing the filter isn't enough — the backend must also parse the format correctly, or you get a 500 instead of success.
- Double-encoding works because filter and request-maker decode a different number of times.
- Bypass technique lists cover a whole vulnerability class — not every one applies to every lab; test to find which fits.
- Isolate one variable at a time to tell which part of a request (host vs path) is causing a block.
