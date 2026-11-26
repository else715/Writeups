# CafeBakes — OTP Bypass

## Title

**OTP Verification Bypass in Authentication Flow**

## Target

**CafeBakes**

## Vulnerability Type

- Authentication Bypass
- OTP Verification Bypass
- Improper Authentication
- Broken Authentication Logic
- CWE-287: Improper Authentication
- CWE-602: Client-Side Enforcement of Server-Side Security

## Severity / CVSS Rationale

**Suggested severity: High**

OTP verification is an authentication security control. If an attacker can reach a verified state without possessing the valid OTP, the weakness may undermine the security of the affected authentication flow.

**Indicative CVSS v3.1: 8.1 (High)**

The exact score depends on:

- whether authentication is required,
- whether an account identifier is needed,
- whether account takeover is possible,
- what functionality becomes available after bypass,
- whether additional authentication controls remain in place.

The final score should therefore be calculated against the exact affected workflow.

---

## Summary

During authorized security testing of the CafeBakes application, an **OTP verification bypass** was identified.

The intended authentication flow required a user to provide a valid OTP after initiating an authentication operation.

However, the verification state could be reached without successfully completing the intended OTP validation.

This indicated that the application was not sufficiently enforcing the OTP requirement on the server side.

The core issue was not the OTP generation itself, but the application's handling of the **verified authentication state**.

---

## Technical Root Cause

A secure OTP architecture should look like:

```text
User
 |
 | username / phone / email
 v
Authentication server
 |
 | generate OTP
 v
OTP delivery
 |
 | OTP
 v
User
 |
 | OTP
 v
Server-side validation
 |
 +---- valid ----> authenticated session
 |
 +---- invalid --> authentication failure
```

The vulnerable architecture effectively allowed the verification state to be influenced without completing the required validation.

Conceptually:

```text
Client
   |
   | verification request/state
   v
Backend
   |
   | insufficient validation
   v
Verified state
```

Authentication status must be created exclusively by trusted server-side logic.

---

## Prerequisites

For authorized testing:

1. A test account.
2. Access to the application's legitimate authentication flow.
3. An HTTP interception tool such as Burp Suite.
4. Permission to test authentication functionality.
5. A controlled account or test environment.

No other user's account should be accessed.

---

## Detailed Reproduction Methodology

### Step 1 — Start authentication

Initiate the normal login/verification workflow.

The application sends an OTP to the registered destination.

Conceptually:

```text
User identifier
      ↓
OTP generated
      ↓
OTP delivered
      ↓
OTP verification page
```

---

### Step 2 — Capture the verification request

Using an authorized interception proxy, observe the request responsible for validating the OTP.

A sanitized example:

```http
POST /[REDACTED]/verify HTTP/2
Host: [REDACTED]
Content-Type: application/json

{
  "user_id": "[REDACTED]",
  "otp": "[REDACTED]"
}
```

The actual endpoint and parameters should not be published if the disclosure program prohibits them.

---

### Step 3 — Analyze the verification state

Determine how the application decides that OTP verification has succeeded.

Important questions include:

- Is the OTP actually checked server-side?
- Is verification tied to the correct user?
- Is the OTP tied to the current authentication transaction?
- Can a client-controlled verification flag influence the result?
- Is the verified state established before validation completes?

---

### Step 4 — Perform controlled validation

Within the authorized test scope, manipulate the relevant verification state or request.

For example, if the application exposes a verification state:

```json
{
  "verified": false
}
```

the test should determine whether changing client-controlled state can cause the server to treat the account as verified.

Do not use another user's account or attempt to access real customer information.

---

### Step 5 — Observe the result

A secure server should reject an invalid or missing OTP.

If the application instead establishes an authenticated or verified session, the OTP control has been bypassed.

---

## Request / Response Example

### Sanitized verification request

```http
POST /[REDACTED]/otp/verify HTTP/2
Host: [REDACTED]
Content-Type: application/json

{
  "identifier": "[REDACTED]",
  "otp": "[REDACTED]"
}
```

### Example of an insecure response

```http
HTTP/2 200 OK
Content-Type: application/json

{
  "success": true,
  "verified": true
}
```

The example is intentionally sanitized.

Do not publish:

- real OTPs,
- session cookies,
- authorization headers,
- phone numbers,
- email addresses,
- customer IDs,
- authentication tokens.

---

## Expected vs. Actual Behavior

### Expected

For an invalid OTP:

```text
OTP submitted
      ↓
Server validates OTP
      ↓
OTP invalid
      ↓
Authentication rejected
```

For a valid OTP:

```text
OTP submitted
      ↓
Server validates OTP
      ↓
OTP valid
      ↓
Verified state created
      ↓
Authenticated session
```

### Actual

Under the tested conditions, the application could reach the verification/authentication state without completing the intended OTP validation.

This demonstrated a failure in the server-side authentication boundary.

---

## Security Impact

Depending on the affected workflow, an OTP bypass can potentially result in:

- Authentication bypass.
- Unauthorized account access.
- Account takeover.
- Circumvention of a second authentication factor.
- Unauthorized access to protected functionality.
- Abuse of account recovery.

The most severe impact occurs when OTP verification is the only barrier between an attacker and an existing account.

---

## Attack Scenario

A simplified attack path is:

```text
Attacker knows account identifier
          ↓
Starts authentication flow
          ↓
OTP verification required
          ↓
Verification mechanism manipulated
          ↓
Server incorrectly establishes verified state
          ↓
Authenticated session
          ↓
Protected functionality
```

The exact prerequisites and impact should be stated according to what was actually demonstrated during the original bug-bounty test.

---

## Remediation

### 1. Validate OTP exclusively server-side

The backend must independently verify:

```text
OTP
+
User
+
Authentication transaction
+
Purpose
+
Expiration
```

### 2. Bind OTP to authentication transaction

An OTP generated for one authentication attempt should not be reusable for another.

### 3. Enforce expiration

OTP values should expire after a short period.

### 4. Enforce single use

Once successfully used, an OTP must immediately become invalid.

### 5. Do not trust client verification flags

Values such as:

```json
{
  "verified": true
}
```

must never be treated as proof of successful OTP validation when supplied by the client.

### 6. Rate-limit verification attempts

Implement controls against:

- brute force,
- OTP guessing,
- automated verification attempts.

### 7. Invalidate previous OTPs

When a new OTP is generated, previous active OTPs should normally be invalidated.

### 8. Create sessions only after verification

The authenticated session should be created only after the server has successfully completed OTP verification.


## Lessons Learned

OTP is only effective when the **server controls the authentication state**.

A strong OTP algorithm cannot protect an application if the backend allows the client to influence whether OTP verification has succeeded.

The key principle is:

```text
Client → requests verification
Server → validates verification
Server → creates authenticated state
```

Never:

```text
Client → declares verification succeeded
Server → trusts client
```
