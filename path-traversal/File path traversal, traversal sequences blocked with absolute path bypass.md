## Path Traversal - Absolute Path Bypass

**Lab:** File path traversal, bypassing absolute path restriction

Application blocks `../` sequences but doesn't validate absolute paths.

### The Defense vs Absolute Paths

**Previous lab (no defense):**

filename=../../../etc/passwd ✓ Works (relative path)


**This lab (defense blocks ../):**

filename=../../../etc/passwd ✗ Blocked
filename=/etc/passwd ✓ Works (absolute path)


### Why Absolute Paths Work

- Defense strips `../` sequences
- Doesn't check for absolute paths starting with `/`
- `/etc/passwd` goes directly to root, doesn't use `../`
- Bypasses the defense entirely

### The Exploit

Instead of relative traversal, use absolute path:

GET /image?filename=/etc/passwd


Server loads the absolute path directly.

### Prevention

Whitelist allowed files. Don't accept ANY file path (relative or absolute):
