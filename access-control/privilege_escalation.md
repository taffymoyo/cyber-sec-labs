## Access Control - Method-based access control can be circumvented

### what i learned
Access controls that only restrict specific HTTP methods can often be bypassed by using a different method that reaches the same functionality.

### the vulnerability
The application enforces authorization checks on one request method (e.g. POST) but not another (e.g. GET).

### how it works
- administrator performs an action using:
  ```
  POST /admin-roles
  ```
- server checks if the user is an administrator
- attacker changes the request method:
  ```
  GET /admin-roles
  ```
- server processes the request without applying the same authorization check

### why it's vulnerable
1. inconsistent access control
   - POST requests require administrator privileges
   - GET requests reach the same endpoint without proper checks

2. authorization tied to request method
   - security logic only exists for one HTTP method
   - switching methods bypasses the restriction

### the exploit

Original request:

```http
POST /admin-roles HTTP/1.1

username=carlos&action=upgrade
```

Modified request:

```http
GET /admin-roles?username=carlos&action=upgrade HTTP/1.1
```

If the server performs the action without checking permissions, the attacker bypasses access control.

### key lesson
Access control should:
1. be enforced regardless of HTTP method
2. validate user permissions before every sensitive action
3. never assume changing the request method makes an action safe
4. apply consistent authorization across all endpoints
