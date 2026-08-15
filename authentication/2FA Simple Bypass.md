## 2FA Simple Bypass

**Vulnerability:** Server doesn't enforce 2FA completion before accessing protected pages.

**How it works:**
- User logs in with username/password
- Server redirects to /verify-2fa
- User manually navigates to /my-account instead
- Server loads page without checking if 2FA was completed

**The exploit:**
1. Log in as victim
2. At 2FA prompt, manually navigate to /my-account in URL
3. Access granted without 2FA code

**Why it's vulnerable:**
Server validates 2FA only on /verify-2fa endpoint, not on protected pages.

**Prevention:**
- Track 2FA status in session
- Check on every protected page: if not 2FA verified, redirect to verification
- Enforce at every step, don't assume users follow intended flow
