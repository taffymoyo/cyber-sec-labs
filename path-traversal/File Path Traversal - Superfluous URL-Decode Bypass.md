## File Path Traversal - Superfluous URL-Decode Bypass

**Lab:** File path traversal, traversal sequences stripped with superfluous URL-decode

App blocks input containing path traversal sequences (`../`), then performs an *extra* URL-decode on the input afterward, before actually using it to fetch the file.

### The Vulnerability

```
filename=../../../etc/passwd            ✗ blocked (contains ../, rejected outright)
filename=..%252f..%252f..%252fetc/passwd ✓ 200 (contents of /etc/passwd returned!)
```

The filter checks the input for `../` and rejects it if found — but only checks the input as it currently is, *before* an additional URL-decode step the app performs afterward. Double-encoding the traversal sequence hides it from that initial check, but the app's own extra decode pass reveals it again.

### The Attack

1. Intercept a request that fetches a product image, send to Repeater
2. Try plain traversal `../../../etc/passwd` → blocked immediately, filter rejects the request
3. Double-URL-encode the traversal sequences: `/` → `%2f` → `%252f`, giving:
   `..%252f..%252f..%252fetc/passwd`
4. Send → filter checks this string, sees no literal `../` (it's `..%252f`, not `../`) → passes
5. App performs its own URL-decode afterward: `%252f` → `%2f` → `/`, reconstructing `../../../etc/passwd`
6. File access proceeds with the real traversal sequence → `/etc/passwd` contents returned

### Why It's Vulnerable

- Filter validates the input **before** a decoding step the app performs later — checking the wrong "version" of the string
- The app's extra ("superfluous") URL-decode happens *after* validation, giving the attacker one decode layer to hide behind
- Same root cause as classic double-encoding bypasses: a mismatch in how many times different parts of the app decode the same input before using/checking it

### Prevention

- Validate input **after** all decoding/normalization is complete — never check a string that will still be transformed afterward
- Avoid unnecessary/duplicate decoding steps in the request-handling pipeline; each additional decode is another opportunity for a filter to be bypassed
- Reject requests containing any encoded traversal sequences too (not just literal `../`), or canonicalize fully before any checks run
- Validate the *final, resolved* file path against the intended base directory instead of pattern-matching on raw input text

### What I Learnt

- This is functionally the same idea as the double-URL-encoding SSRF bypass, but the mismatch here is deliberately built into the app itself (an unnecessary decode step after validation), not just an accidental byproduct of using different libraries.
- "Superfluous" decode is a good phrase for the bug: the app didn't need to decode again after already receiving decoded input from the web server/framework — that redundant step is exactly what creates the gap for the filter to be bypassed.
- General lesson reinforced again: **validate the fully resolved, final value — never a version of the input that will still change after the check runs.**
