## File Path Traversal - Simple Case

**Lab:** File path traversal, simple case

Application doesn't validate `filename` parameter, allowing attacker to access arbitrary files via `../` sequences.

### The Vulnerability

**Normal request:**

GET /image?filename=productImage.jpg

Returns: product image

**Vulnerable request:**

GET /image?filename=../../../etc/passwd

Returns: /etc/passwd file contents

### The Exploit

1. Intercept image request in Burp
2. Change `filename` parameter to `../../../etc/passwd`
3. Server doesn't sanitize path
4. Traverses up directories and reads passwd file

### Why It's Vulnerable

- No input validation on filename
- Allows `../` sequences (directory traversal)
- Server doesn't restrict file access to intended directory
- Attacker reads arbitrary files with app's permissions

### Prevention

- Whitelist allowed filenames (don't use user input directly)
- Never allow `../` or absolute paths
- Use a fixed directory: `files/` + sanitized filename only
- Example: `String safePath = new File(baseDir, userInput).getCanonicalPath()`

### Key Lesson

Never trust user input for file paths. Always validate against whitelist.
