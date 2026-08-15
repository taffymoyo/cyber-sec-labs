## Authentication Bypass - OAuth Implicit Flow

Application fails to validate OAuth token matches the user identity claimed in the request.

### The Vulnerability

**Legitimate flow:**
User clicks "Login with OAuth"
OAuth service authenticates user
OAuth returns: token + user info (email, username)
App sends to /authenticate: {"email":"wiener@...", "token":"xyz123"}
App validates: Does token xyz123 belong to wiener@...? ✓
App creates session for wiener

**Flawed flow (this lab):**
User clicks "Login with OAuth"
OAuth service authenticates user
OAuth returns: token + user info
App sends to /authenticate: {"email":"wiener@...", "token":"xyz123"}
App checks: Does endpoint exist? ✓ (no token validation)
App trusts the email/username blindly
App creates session for whatever email is sent

### The Exploit

1. Complete OAuth login as wiener
2. Intercept POST /authenticate request in Burp
3. Change email to `carlos@carlos-montoya.net`
4. Server doesn't validate token matches email
5. Logged in as Carlos

**Request:**
```json
{"email":"carlos@carlos-montoya.net","username":"Carlos","token":"wiener's_token"}
```

Server processes it without checking if that token belongs to Carlos.

### Why It's Vulnerable

- Server receives OAuth token
- Server receives email/username
- **Server doesn't validate they match**
- Just trusts whatever email is sent
- No verification that token belongs to that user

### Prevention

- Validate OAuth token on every /authenticate request
- Verify token claims match the email/username sent
- Example: `SELECT user_email FROM oauth_tokens WHERE token='xyz123'`
- If it returns different email, reject the request
- Apply depth-of-defense: validate at login, validate at redirects

### Key Lesson

Never trust user-provided identity claims. Always validate against authoritative source (OAuth token, session database, etc).
