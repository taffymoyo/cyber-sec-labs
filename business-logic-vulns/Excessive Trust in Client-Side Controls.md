## Business Logic - Excessive Trust in Client-Side Controls

**Lab:** Excessive trust in client-side controls

Application trusts hidden form fields sent by the client instead of validating server-side.

### The Vulnerability
```html
<input type=hidden name=productId value=1>
<input type=hidden name=price value=1337>
<input type=hidden name=quantity value=1>
```

Server processes the hidden `price` parameter directly without verifying it matches the database.

### The Exploit
1. Add item to cart
2. Intercept in Burp
3. Modify `quantity=-1` (negative quantity = refund instead of charge)
4. Or modify `price=0.01` (charge almost nothing)
5. Checkout with modified values

Result: Get expensive item for free or nearly free.

### Why It's Vulnerable
- Server assumes hidden fields can't be modified
- No server-side validation of price
- Trusts client-provided quantity without checking if it's positive

### Prevention
Server should:
1. Receive only `productId` and `quantity`
2. Look up real price from database: `SELECT price FROM products WHERE id=productId`
3. Validate quantity > 0
4. Calculate total server-side
5. Charge based on database values, never client input

### Lab Result
✓ Exploited by using negative quantity to refund expensive jacket
✓ Completed checkout with manipulated client-side values
