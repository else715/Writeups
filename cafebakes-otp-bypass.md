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
```
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
```
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

> Reproduction steps omitted from the public version.

---

## Request / Response Example

> Request and response examples omitted from the public version.

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
```
OTP submitted
      ↓
Server validates OTP
      ↓
OTP invalid
      ↓
Authentication rejected
```

For a valid OTP:
```
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

> Attack sequence omitted from the public version.

The exact prerequisites and impact should be stated according to what was actually demonstrated during the original bug-bounty test.

---

## Remediation

### 1. Validate OTP exclusively server-side

The backend must independently verify:
```
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
```
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

---

## Disclosure / Acknowledgement Information

**Disclosure type:** Bug bounty / responsible security research



**Researcher:** Manojkumar Gurram



Add the original program information here if you have it:
```
Reported:
[Insert date]

Report/reference:
[Insert report ID]

Acknowledgement:
[Insert acknowledgement]

Resolution:
[Insert fixed/resolved status]
```

Sensitive authentication material should never be included.

---

## Lessons Learned

OTP is only effective when the **server controls the authentication state**.



A strong OTP algorithm cannot protect an application if the backend allows the client to influence whether OTP verification has succeeded.



The key principle is:
```
Client → requests verification
Server → validates verification
Server → creates authenticated state
```

Never:
```
Client → declares verification succeeded
Server → trusts client
```
