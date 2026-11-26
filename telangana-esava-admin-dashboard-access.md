#  state Government eSava - Admin Dashboard Access

## Title

**Unauthorized Access to Administrative Dashboard Through Broken Access Control**

## Target

**state Government eSava**

## Vulnerability Type

- Broken Access Control
- Vertical Privilege Escalation
- Improper Authorization
- CWE-862: Missing Authorization
- CWE-863: Incorrect Authorization

## Severity / CVSS Rationale

**Suggested severity: High to Critical depending on confirmed administrative capabilities**

Administrative access can have significant impact because privileged interfaces may expose sensitive functionality and data.

A reasonable starting assessment is:

**Indicative severity: High**

A CVSS score should be based on the exact administrative functionality that was demonstrably accessible.

For example, if the issue allowed unrestricted modification of sensitive records or administrative configuration, the final severity could be higher.

> Do not claim Critical severity solely because an admin dashboard was reachable. The severity should reflect the actual privileges and impact demonstrated.

---

## Summary

During authorized security testing of the state Government eSava application, a **broken access-control vulnerability** was identified.

The application allowed an unauthorized user context to reach functionality intended for administrative users.

The fundamental issue was insufficient server-side enforcement of the user's authorization level.

Administrative functionality should never be protected solely by:

- hidden frontend menus,
- client-side role checks,
- predictable navigation,
- UI restrictions.

The backend must independently verify the user's permissions whenever a privileged resource or operation is requested.

---

## Technical Root Cause

The security boundary should be:

```text
User request
      |
      v
Authentication
      |
      v
Identify user
      |
      v
Determine role/permissions
      |
      v
Authorization check
      |
      +---- Authorized ----> Admin resource
      |
      +---- Unauthorized --> 403 / access denied
```

The affected behavior indicated that the authorization step was insufficient or missing for the relevant administrative resource.

A vulnerable architecture may effectively behave like:

```text
Request
  |
  v
Authentication
  |
  v
Resource
  |
  v
Admin functionality
```

without adequately checking:

```text
Does this user have permission to access this resource?
```

---

## Prerequisites

For responsible testing:

1. An explicitly authorized account or test environment.
2. A defined bug-bounty/security-testing scope.
3. A normal authenticated session where authentication is required.
4. An HTTP interception/testing tool if request-level validation is necessary.
5. Permission to test access-control boundaries.

Because this is a government service, testing should be strictly non-destructive and limited to the minimum necessary to establish the vulnerability.

---

## Detailed Reproduction Methodology

### Step 1 — Establish the normal user role

Authenticate using the authorized test account.

Document the account's intended role:

```text
Role:
Non-administrative user
```

Do not use another person's credentials.

---

### Step 2 — Identify an administrative resource

During authorized assessment, identify an administrative function that should not be available to the test account.

The public writeup intentionally does not disclose the live administrative path.

For documentation:

```text
Administrative resource:
[xxxxx]
```

---

### Step 3 — Test server-side authorization

Request the restricted resource using the authorized lower-privileged account.

A sanitized example:

```http
GET /[xxxxxx]/admin/[xxxxxx] HTTP/2
Host: [xxxxxx]
Cookie: [xxxxxx]
```

The important security question is whether the server checks the user's authorization before returning the administrative resource.

---

### Step 4 — Observe the response

A secure implementation should respond with something similar to:

```http
HTTP/2 403 Forbidden

{
  "error": "Access denied"
}
```

If the server instead returns the administrative interface or privileged functionality, this indicates an authorization failure.

---

### Step 5 — Stop after confirmation

Once unauthorized access is demonstrated:

- Do not enumerate administrative records.
- Do not download citizen information.
- Do not modify government records.
- Do not create administrative users.
- Do not execute administrative actions.
- Do not attempt persistence.
- Do not attempt to expand access beyond the minimum proof.

This is especially important for government infrastructure.

---

## Request / Response Example

### Sanitized request

```http
GET /[xxxxxx]/admin/dashboard HTTP/2
Host: [xxxxxx]
Cookie: session=[xxxxxx]
Authorization: [xxxxxx]
```

### Expected response

```http
HTTP/2 403 Forbidden
Content-Type: application/json

{
  "error": "Unauthorized"
}
```

### Observed response

```http
HTTP/2 200 OK
Content-Type: text/html

[ADMIN DASHBOARD CONTENT REDACTED]
```

The response shown here is a sanitized representation.

Do not publish:

- real session cookies,
- authorization tokens,
- government records,
- citizen information,
- employee information,
- internal server information,
- sensitive endpoint details.

---

## Expected vs. Actual Behavior

### Expected

A non-administrative user should receive an authorization failure:

```text
Authenticated user
       ↓
Role = normal user
       ↓
Request admin resource
       ↓
Authorization check
       ↓
DENIED
       ↓
403 Forbidden
```

### Actual

The tested application allowed the unauthorized context to reach administrative dashboard functionality:

```text
Authenticated user
       ↓
Role = normal user
       ↓
Request admin resource
       ↓
Insufficient authorization enforcement
       ↓
ADMIN RESOURCE
```

This demonstrates a broken privilege boundary.

---

## Security Impact

Administrative access may potentially expose:

- privileged functionality,
- sensitive application information,
- administrative records,
- user-management capabilities,
- configuration functionality,
- data-management operations.

The actual impact should be documented according to what was safely and explicitly demonstrated.

For example:

> The vulnerability allowed access to the administrative dashboard interface from a non-administrative user context. No destructive actions or unauthorized modification of government records were performed during validation.

This is preferable to claiming broader compromise that was not demonstrated.

---

## Attack Scenario

A simplified attack scenario is:

```text
Attacker obtains ordinary authenticated account
             |
             v
Normal user privileges
             |
             v
Requests restricted administrative resource
             |
             v
Server fails authorization check
             |
             v
Administrative dashboard exposed
             |
             v
Potential access to privileged functionality
```

If the vulnerable authorization check applies to multiple endpoints, the impact could extend beyond the dashboard itself.

However, each additional privilege should be demonstrated independently and responsibly.

---

## Remediation

### 1. Enforce authorization server-side

Every administrative endpoint must perform an authorization check.

```text
Request
  ↓
Authentication
  ↓
Authorization
  ↓
Resource
```

### 2. Deny by default

Users should receive access only when explicitly authorized.

```text
No permission
    ↓
DENY
```

rather than:

```text
No permission
    ↓
Maybe allow
```

### 3. Centralize authorization

Use a consistent authorization mechanism instead of implementing ad-hoc checks across individual controllers or endpoints.

### 4. Do not trust frontend roles

A client-side value such as:

```javascript
isAdmin = true
```

must never determine whether the backend permits administrative access.

### 5. Validate every privileged operation

Authorization must be enforced not only on:

```text
GET /admin/dashboard
```

but also on operations such as:

```text
create
update
delete
approve
export
manage users
change configuration
```

where applicable.

### 6. Implement automated authorization testing

Test combinations such as:

```text
Normal user → admin resource
Admin → normal resource
User A → User B resource
Expired session → protected resource
Unauthenticated → protected resource
```

### 7. Review related endpoints

A broken authorization pattern can affect multiple administrative endpoints.

A comprehensive remediation should therefore review the entire privilege boundary rather than patching only one URL.

---

## Lessons Learned

The most important lesson is:

> **Authentication answers "Who are you?" Authorization answers "What are you allowed to do?"**

A user can be legitimately authenticated and still have no permission to access an administrative resource.

Administrative interfaces should therefore be protected by server-side authorization at every resource and operation.

Frontend restrictions are useful for user experience, but they are not security controls.

For government applications in particular, broken access control should be treated seriously because the potential confidentiality, integrity, and public-service consequences can be significant.


Testing government infrastructure should always remain within explicit authorization and responsible-disclosure boundaries.
