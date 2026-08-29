## File Path Traversal - Null Byte Bypass of File Extension Check

**Lab:** File path traversal, validation of file extension with null byte bypass

App validates that the supplied `filename` parameter *ends with* an expected extension (e.g. `.png`) before serving product images. A null byte in the filename lets an attacker satisfy that check while the actual file access ignores everything after it.

### The Vulnerability

```
filename=../../../etc/passwd            ✗ blocked (doesn't end in .png)
filename=../../../etc/passwd%00.png     ✓ 200 (contents of /etc/passwd returned!)
```

The extension check reads the whole string and sees it ends in `.png` → passes. But the underlying file-open call (which relies on null-terminated strings at a lower level) stops reading at the null byte — so it actually opens `../../../etc/passwd`, ignoring `.png` entirely.

### The Attack

1. Intercept a request that fetches a product image, send to Repeater
2. Modify the `filename` parameter to:
   `../../../etc/passwd%00.png`
3. Send the request
4. Response returns the contents of `/etc/passwd`

### Why It's Vulnerable

- Extension validation only checks that the string *ends with* the expected extension — a surface-level string check
- Null byte (`%00`) is treated as "end of string" by lower-level file-handling APIs (legacy C-based convention), but not by the higher-level validation code that reads the full string
- Mismatch between what the validator "sees" and what the file-open call actually uses as the filename — same class of bug as the double-decoding SSRF path/host bypasses (two components interpreting the same input differently)
- Path traversal (`../../../`) is still present underneath — the null byte trick only defeats the extension check, it doesn't cause the traversal itself

### Prevention

- Never rely on languages/libraries with null-terminated string handling for security-critical validation without accounting for early termination
- Validate and canonicalize the *resolved* file path, and confirm it stays within the intended base directory, rather than checking surface properties of the raw string (prefix/suffix)
- Reject filenames containing null bytes (`%00`, `\x00`) outright
- Map user input to a whitelist of known filenames/IDs instead of accepting a raw path + extension from the client
- Keep underlying language runtimes/libraries updated — many modern runtimes no longer truncate strings on null bytes, closing off this specific bypass

### What I Learnt

- A null byte (`\x00`) signals "end of string" in older/lower-level string handling (C, and anything built on it) — but higher-level app code often reads the *entire* string, creating a gap between what's validated and what's actually used.
- This bug combines two separate issues: path traversal (`../../../`) gets you out of the intended directory, and the null byte (`%00.png`) is purely there to slip past the extension check — neither one alone solves the lab.
- Same root pattern as other bypasses I've seen: whenever two different components (validator vs. actual file/URL/request handler) parse the same input differently, that gap is exploitable.
- Extension-only or suffix-only checks are inherently weak, since they check what the string looks like, not what the underlying system will actually resolve/open.
