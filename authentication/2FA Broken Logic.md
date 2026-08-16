## 2FA Broken Logic

**Lab:** 2FA broken logic

Application uses client-controlled `verify` parameter to determine which user's 2FA code is checked. No rate limiting on verification attempts allows brute force.

### The Vulnerability

**2FA verification request:**

POST /login2
verify=wiener&mfa-code=1234


**Flaw:** The `verify` parameter controls which user is authenticated, not the session.

Attacker changes:

POST /login2
verify=carlos&mfa-code=1234


Server verifies the code against **carlos's account**, not the logged-in user.

### The Exploit

1. GET /login2 with `verify=carlos` → generates 2FA code for carlos
2. POST /login2 with `verify=carlos&mfa-code=0000`
3. Brute force mfa-code from 0000-9999 (10000 attempts)
4. No rate limiting → eventually correct code found
5. Logged in as carlos

### Why It's Vulnerable

- `verify` parameter is client-controlled (should be server-side session)
- No rate limiting on failed attempts
- No account lockout after X failures
- 4-digit code = only 10000 possibilities

### Prevention

- Never trust client-provided user identifier
- Use session to track authenticated user
- Implement rate limiting: lock after 3-5 failed attempts
- Add exponential backoff between attempts
- Log failed attempts for security monitoring

### Note

Burp Community throttles Intruder (slow). Requires Pro for efficient brute force of 10000 requests.
