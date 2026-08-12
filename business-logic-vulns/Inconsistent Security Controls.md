## Inconsistent Security Controls

**Vulnerability:** Server validates email domain inconsistently across different features.

**How it works:**

1. **Registration:** Server allows ANY email during signup
   - `anything@web-security-academy.net` ✓ Accepted
   - No domain check

2. **Email change:** Server allows changing to ANY domain after registration
   - Change to `attacker@dontwannacry.com` ✓ Accepted
   - No validation that domain matches company

3. **Admin access:** Server grants access based on email domain
   - User has `@dontwannacry.com` email → Admin access ✓

**The exploit:**
1. Register with arbitrary email at exploit server domain
2. Confirm via email client
3. Change email to `@dontwannacry.com` 
4. Server sees `@dontwannacry.com` → grants admin access
5. Delete carlos

**Why it's vulnerable:**
- Registration doesn't validate you work there
- Email change doesn't validate domain
- Access control trusts email domain (which you just changed)
- No consistency: should validate domain on registration OR block email changes

**Prevention:**
- Validate email domain on registration (reject non-company emails)
- OR block email changes to different domains
- OR verify ownership of domain before granting access
- Apply same rules everywhere, not just one place
