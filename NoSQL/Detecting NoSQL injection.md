## NoSQL Injection - Detecting and Exploiting via Boolean Conditions

**Lab:** Detecting NoSQL injection

App builds a NoSQL (JavaScript-based) query using unsanitized user input from the `category` parameter. Injecting JavaScript syntax lets an attacker manipulate the query logic to bypass intended filtering.

### The Vulnerability

```
category=Gifts                    -> normal response, Gifts category products shown
category=Gifts'                   -> JavaScript syntax error (unbalanced quote)
category=Gifts'+'                 -> no error (string concatenation is valid JS)
category=Gifts' && 0 && 'x        -> no products (false condition injected)
category=Gifts' && 1 && 'x        -> Gifts products shown (true condition injected)
category=Gifts'||1||'             -> ALL products shown, including unreleased ones
```

The app appears to insert the `category` value directly into a server-side JavaScript/NoSQL query string without sanitizing quotes or logical operators — allowing injected JS expressions to alter the query's evaluated logic.

### The Attack

1. Click a product category filter, capture the request, send to Repeater
2. Submit `'` alone in `category` -> triggers a JS syntax error -> signals unsanitized input reaching a JS/NoSQL context
3. Submit `Gifts'+'` (URL-encoded) -> no error -> confirms input is being concatenated into a JS expression (string concatenation is valid JS)
4. Test boolean control:
   - `Gifts' && 0 && 'x` -> no results (forces query condition false)
   - `Gifts' && 1 && 'x` -> normal results (forces query condition true)
   - This confirms you can control the truthiness of the underlying query logic
5. Submit `Gifts'||1||'` -> forces the whole condition to always evaluate true, regardless of category -> returns all products, including ones not meant to be publicly listed
6. View the response in browser to confirm unreleased products are exposed

### Why It's Vulnerable

- User input is concatenated directly into a server-side query/expression without sanitization or parameterization
- No escaping of special characters (`'`, `&&`, `||`) that have syntactic meaning in the query language
- The query's logic can be altered by injecting operators that always evaluate to true, bypassing intended filtering conditions entirely

### Prevention

- Use parameterized queries / prepared statements for NoSQL queries, just as with SQL — never concatenate raw user input into a query string
- Strictly validate and sanitize input (e.g., reject unexpected characters like quotes and logical operators in fields that should only contain simple identifiers)
- Apply the principle of least privilege: queries should not be able to return data outside their intended scope regardless of injected conditions
- Use an allowlist of valid category values instead of accepting arbitrary user-controlled strings

---

### ⚠️ Personal note (not for tomorrow)

I flagged that my understanding of *why* this is called NoSQL injection, and how it differs mechanically from SQL injection, is still shaky. Concepts I want to solidify before doing another related lab:
- What "NoSQL" actually means and how these queries are typically structured (e.g., MongoDB-style vs. this JS-expression style)
- Why unbalanced quotes cause a JS syntax error here specifically (what's the underlying query construction look like server-side?)
- Why `&&` and `||` work as injection levers — what's actually being evaluated line-by-line
- How this compares/contrasts with classic SQL injection (similar boolean-based technique, different underlying language/engine)

**Reminder for next session: don't help me start or solve another lab, and don't write another git/markdown write-up, until I've worked through and can explain the NoSQL injection concept myself.**
