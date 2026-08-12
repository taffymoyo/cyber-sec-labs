## Inconsistent Security Controls

Application applies different validation rules to different request paths. One endpoint validates quantity strictly, another doesn't.

**Exploit:** Bypass validation on the less-protected endpoint. Send negative quantities or invalid values where one path checks, another doesn't.

**Lesson:** Security controls must be consistent across all endpoints. Don't assume some paths are "less sensitive."
