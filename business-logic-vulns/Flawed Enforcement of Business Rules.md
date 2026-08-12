## Flawed Enforcement of Business Rules

**Vulnerability:** Server doesn't properly track coupon/discount usage. Validates the coupon EXISTS, but not whether USER ALREADY USED IT.

**How it works:**

User applies coupon: "PROMO50"
Server checks: Does PROMO50 exist? Yes ✓
Server checks: Is PROMO50 expired? No ✓
Server checks: Has THIS USER already used PROMO50? [Missing check]
Server applies discount

User applies PROMO50 again:
Same checks pass (no tracking of per-user usage)
Discount applied again


**Why coupons can be reused:**
- Server validates: code exists, code is valid, code not expired
- Server does NOT validate: has this user already claimed it?
- No database table tracking `user_id → coupon_code → redeemed`
- Each request is treated independently, no state checking

**Example exploit:**

Request 1: POST /apply-coupon code=PROMO50
Response: Discount applied, total $50 → $25

Request 2: POST /apply-coupon code=PROMO50
Response: Discount applied again, total $25 → $12.50

Request 3: POST /apply-coupon code=PROMO50
Response: Discount applied again, total $12.50 → $6.25


**Prevention:**
- Track coupon redemption in database
- Check: `SELECT * FROM coupon_usage WHERE user_id=X AND coupon_code='PROMO50'`
- If exists, reject
- Only allow one redemption per user per coupon
- Track state server-side, never trust client

Commit this version instead. More detail, more learning.


