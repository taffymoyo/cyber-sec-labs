# Access Control - Method-based access control and multi-step process bypasses

## What I learned

Access control vulnerabilities happen when authorization checks are applied inconsistently. This can occur when security depends on the HTTP method or when only some steps of a multi-step process are protected.

---

## Scenario 1 - Method-based access control can be circumvented

### The vulnerability

The application checks authorization for one HTTP method (such as `POST`) but not another (such as `GET`).

### Example

Original request:

```http
POST /admin-roles HTTP/1.1

username=carlos&action=upgrade
```

Modified request:

```http
GET /admin-roles?username=carlos&action=upgrade HTTP/1.1
```

If the server performs the action without checking permissions, access control is bypassed.

### Key point

Authorization should be enforced regardless of the HTTP method used.

---

## Scenario 2 - Access control vulnerabilities in multi-step processes

### The vulnerability

Some sensitive actions require multiple steps, for example:

1. Load the edit form.
2. Submit changes.
3. Confirm the changes.

If only the first steps check permissions, an attacker may skip directly to the final step.

### Key point

Every step that performs a sensitive action should verify the user's permissions.

---

## Scenario 3 - Missing access control on one step

### The vulnerability

A confirmation page is assumed to only be reachable through the admin panel, so it does not perform its own authorization check.

### Example

Using the session ID for the low-privileged user **Wiener** (ending in `1c`), the confirmation ("Are you sure?") page could be accessed directly.

Because this step did not authenticate the session, the privileged action was completed successfully.

### Key point

Never assume users will follow the intended workflow. Every request should perform its own authorization check.

---

## Key lessons

* Enforce access control on every request.
* Don't rely on HTTP methods for security.
* Check permissions at every step of a multi-step process.
* Never assume a page is only reachable through the intended workflow.
