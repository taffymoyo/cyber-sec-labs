## Race Condition - Limit Overrun

**Lab:** Limit overrun race conditions

Application processes requests sequentially but doesn't handle simultaneous requests properly.

### The Vulnerability

**Normal flow:**

Request 1: Apply coupon "PROMO50"
→ Check: Coupon used before? No
→ Apply discount
→ Update: coupon_usage table
→ Return: Discount applied

Request 2: Apply coupon "PROMO50"
→ Check: Coupon used before? Yes (from request 1)
→ Reject


**Race condition flow:**

Request 1 + Request 2 sent simultaneously

Request 1: Check coupon_usage... (slow)
Request 2: Check coupon_usage... (same slow check, both see "not used")
Request 1: Apply discount, update table
Request 2: Apply discount, update table

Result: Coupon applied twice before either could update


### The Exploit

1. Send multiple identical requests simultaneously (tab groups in Burp Repeater)
2. Requests race to reach server at same microsecond
3. Server's validation hasn't updated yet
4. Multiple requests pass validation before state updates
5. Coupon applies multiple times (or balance depletes multiple times)

### Why It's Vulnerable

- Server checks state (coupon used? balance sufficient?)
- Server processes request (apply coupon, deduct balance)
- **Gap between check and update**
- If two requests hit this gap simultaneously, both see old state

### Why Some Succeed, Some Fail

Timing is chaotic:
- Request arrives at microsecond 0.0001 → Might succeed
- Request arrives at microsecond 0.0002 → Might fail
- No guarantee which wins the race
- That's why it's unreliable (but works sometimes)

### Prevention

- Use database transactions (atomic operations)
- Lock resources: `BEGIN TRANSACTION ... COMMIT`
- Ensures check + update happens as one indivisible operation
- No race window possible

### Lab Result

✓ Exploited by sending multiple coupon requests simultaneously
✓ Tab groups in Repeater used (Intruder too slow in Community)
✓ Race condition collision caused multiple coupons to apply
