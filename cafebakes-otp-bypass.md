# CafeBakes — OTP Verification Bypass

**Researcher:** Manojkumar Gurram  
**Classification:** Improper Authentication (CWE-287)

## Reported finding

During authorized testing of the CafeBakes authentication flow, the researcher reported that a test account reached a verified state before the server had successfully validated its one-time password (OTP). The finding concerns the server’s decision to mark an authentication attempt as verified.

## Potential impact

A verified session could grant access to features protected by the OTP check. The report describes the verification bypass on a test account. Access to another customer’s account and the scope of any protected functionality remain unconfirmed. The researcher proposed High severity, pending confirmation of those consequences.

## Remediation

- Keep the authentication attempt pending until the server validates the OTP for the correct user, transaction, and purpose. Only the server should grant verified status.
- Enforce expiration and single use. Consume the OTP as part of successful verification, and invalidate the previous code when issuing a replacement for the same attempt.
- Rate-limit verification attempts. Issue or upgrade the session only after all required authentication checks succeed.

## Disclosure status

The supplied report lists the disclosure as bug bounty research. The reporting date, vendor acknowledgement, and remediation status are undocumented.
