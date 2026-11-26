# Telangana Government eSava — Administrative Dashboard Access

**Researcher:** Manojkumar Gurram  
**Classification:** Broken access control

## Reported finding

During an authorized assessment of the Telangana Government eSava application, the researcher reported that the application displayed an administrative dashboard to a test account with an ordinary user role. Viewing the dashboard required administrative permission under the intended access policy.

## Scope and impact

The report documents access to the dashboard interface. Whether the account could read citizen records, change data, or carry out administrative actions remains unconfirmed. The report leaves the final severity open because it depends on which data and operations the account could access.

## Remediation

- Apply a shared server-side authorization policy to the dashboard and each underlying operation. Deny access unless the account has the permission required for that resource or action.
- Check permissions before returning protected data or applying changes. Hiding a menu item only changes what the interface displays.
- Add regression tests covering ordinary users, administrators, and expired sessions. Check both the access decision and its effects: a denied operation should return no protected data and leave records unchanged.

## Disclosure status

The supplied report lists responsible vulnerability disclosure as its disclosure type. The report date, acknowledgement, and remediation status are undocumented.
