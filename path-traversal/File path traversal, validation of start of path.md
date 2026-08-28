## File Path Traversal - Bypassing "Start of Path" Validation

**Lab:** File path traversal, validation of start of path

App checks that the user-supplied `filename` parameter *starts with* an expected base folder (e.g. `/var/www/images`), assuming this guarantees the file stays inside that directory. It doesn't account for traversal sequences appearing later in the same string.

### The Vulnerability

```
filename=/var/www/images/58.jpg                                  ✓ 200 (normal image)
filename=../../../etc/passwd                                     ✗ blocked (doesn't start with base folder)
filename=/var/www/images/../../../etc/passwd                     ✓ 200 (traversal!)
```

The validation only checks the prefix of the string. As long as the required base folder appears at the start, everything after it — including `../` sequences that walk back out of that folder — is passed straight to the filesystem.

### The Attack

1. Find a request that loads a file via a `filename` (or similar) parameter, send to Repeater
2. Try plain traversal (`../../../etc/passwd`) → blocked, since it doesn't start with the expected base path
3. Prepend the expected base folder to the traversal sequence:
   `filename=/var/www/images/../../../etc/passwd`
4. Send → passes the "starts with" check, but the OS still resolves the `../` sequences, walking out of `/var/www/images` and up to root
5. Response returns the contents of `/etc/passwd`

### Why It's Vulnerable

- Validation only checks that the string *starts with* a fixed prefix — a purely textual/positional check
- No check on what comes *after* the prefix, so traversal sequences later in the path are untouched
- The filesystem resolves `../` sequences normally regardless of what prefix preceded them — "starts with the right folder" says nothing about where the path *ends up*

### Prevention

- Don't validate by string prefix alone — resolve the full path first (canonicalize), then check that the *resolved* path is still inside the intended base directory
- Strip or reject any `../`, `..\`, encoded variants (`%2e%2e/`), or null bytes before use
- Use platform APIs that map a safe identifier to a file (e.g. a whitelist of known filenames/IDs) rather than accepting a raw path from the user
- Run the file-serving process with least-privilege filesystem permissions, so even a successful traversal can't reach sensitive files

### What I Learnt

- "Starts with X" is a weak check — it validates position, not final destination. An attacker can satisfy the prefix requirement and still smuggle traversal sequences right after it.
- Path canonicalization (resolving `../` etc.) happens at the OS/filesystem level, which doesn't care what the developer's string check already approved — validation needs to happen *after* resolving the path, not just on the raw input text.
- This is the same theme as the SSRF blacklist bypasses: checking surface-level string properties (starts-with, contains, doesn't-contain) is fundamentally weaker than checking the actual resolved value.
