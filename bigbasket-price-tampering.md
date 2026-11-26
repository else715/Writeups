# BigBasket - Price Tampering

## Title

**Client-Side Price Manipulation Leading to Unauthorized Transaction Price Modification**

## Target

**BigBasket**

## Vulnerability Type

- Business Logic Vulnerability
- Improper Server-Side Validation
- Price Manipulation
- CWE-602: Client-Side Enforcement of Server-Side Security
- CWE-20: Improper Input Validation

## Severity / CVSS Rationale

**Suggested severity: High**

A price manipulation vulnerability can have a direct financial impact because an attacker may influence the amount associated with a purchase.

A reasonable CVSS assessment depends heavily on the exact transaction flow and whether authentication or other conditions are required.

**Indicative CVSS v3.1: 8.1 (High)**

Example rationale:

- **Attack Vector:** Network
- **Attack Complexity:** Low
- **Privileges Required:** Low/None depending on the affected flow
- **User Interaction:** None
- **Scope:** Unchanged
- **Confidentiality:** None
- **Integrity:** High
- **Availability:** None

> The final CVSS score should be recalculated against the exact vulnerable endpoint and business impact demonstrated during the original assessment.

---

## Summary

During security testing of the BigBasket application, a **price-tampering vulnerability** was identified in the purchase workflow.

The application exposed a price-related value to the client and the backend did not sufficiently ensure that the final transaction amount originated from an authoritative server-side price.

By manipulating the relevant value during an authorized test, it was possible to influence the price used by the transaction flow.

The fundamental security problem was that a **financially sensitive value was being trusted from a client-controlled context**.

A price displayed by the frontend should never be considered authoritative.

The server should independently retrieve the current product price and calculate the final payable amount.

---

## Technical Root Cause

The underlying issue was a failure to maintain a proper trust boundary between the client and server.

A simplified insecure architecture looks like:

```text
Client
   |
   | product_id
   | quantity
   | price = client-controlled value
   v
Backend
   |
   | accepts price
   v
Order calculation
```

The secure architecture should instead be:

```text
Client
   |
   | product_id
   | quantity
   v
Backend
   |
   | retrieves authoritative price
   v
Server-side calculation
   |
   | validates discounts
   |
   | calculates final total
   v
Payment
```

The client can provide a product identifier and quantity, but the server should determine the authoritative price.

---

## Prerequisites

For an authorized security assessment, the following were sufficient:

1. A legitimate application account where required.
2. Access to the normal shopping/cart workflow.
3. An intercepting HTTP proxy such as Burp Suite or an equivalent testing tool.
4. A test product/order flow within the permitted bug-bounty scope.
5. Permission to modify application requests.

No attempt should be made to complete an unauthorized purchase or cause financial loss.

---

## Detailed Reproduction Methodology

### Step 1 — Identify a product

Select a product through the normal application interface.

For example:

```text
Product:
Example Product

Displayed price:
₹999
```

The exact product and values used during the original assessment should be documented separately if they are safe and authorized for disclosure.

---

### Step 2 — Add the product to the cart

Add the product to the shopping cart.

Observe the information exchanged between the client and server.

Conceptually, the application may send information similar to:

```http
POST /[REDACTED]/cart HTTP/2
Host: [REDACTED]

{
  "product_id": "[REDACTED]",
  "quantity": 1,
  "price": 999
}
```

The above is a **sanitized representation**, not the original production request.

---

### Step 3 — Inspect the request

Using an authorized interception proxy, inspect the request and determine whether the price is:

- calculated entirely on the server, or
- transmitted by the client and subsequently trusted.

The important security question is:

> Does the backend independently determine the price associated with `product_id`?

---

### Step 4 — Modify the price value

During an authorized test, modify the relevant price value.

For example:

```json
{
  "product_id": "[REDACTED]",
  "quantity": 1,
  "price": 999
}
```

could be changed in the controlled test to:

```json
{
  "product_id": "[REDACTED]",
  "quantity": 1,
  "price": 99
}
```

The actual values from the original test should only be included if they are safe to disclose.

---

### Step 5 — Submit the modified request

Forward the request and observe the server's response.

A secure implementation would recalculate the amount using authoritative backend data.

An affected implementation may instead reflect the manipulated amount in the resulting transaction state.

---

### Step 6 — Verify the result safely

The purpose of validation is to establish that the server accepted an unauthorized price modification.

Testing should stop once this behavior has been demonstrated.

There is no need to:

- complete a fraudulent purchase,
- exploit the issue repeatedly,
- target other customers,
- manipulate real financial transactions.

---

## Request / Response Example

### Original request

```http
POST /[REDACTED]/cart/update HTTP/2
Host: [REDACTED]
Content-Type: application/json
Authorization: [REDACTED]

{
  "product_id": "[REDACTED]",
  "quantity": 1,
  "price": 999
}
```

### Modified request

```http
POST /[REDACTED]/cart/update HTTP/2
Host: [REDACTED]
Content-Type: application/json
Authorization: [REDACTED]

{
  "product_id": "[REDACTED]",
  "quantity": 1,
  "price": 99
}
```

### Sanitized response

```http
HTTP/2 200 OK
Content-Type: application/json

{
  "success": true,
  "total": 99
}
```

The values above are illustrative/sanitized. Replace them with your actual evidence only after removing credentials, cookies, tokens, customer information, and other sensitive data.

---

## Expected vs. Actual Behavior

### Expected

The server should ignore any client-controlled price and independently determine:

```text
Product ID
      ↓
Authoritative product record
      ↓
Current price
      ↓
Discount validation
      ↓
Quantity calculation
      ↓
Final payable amount
```

For example:

```text
Client sends:
price = ₹99

Server:
Product price = ₹999

Final server-side price:
₹999
```

### Actual

The tested workflow allowed the client-controlled price value to influence the transaction calculation.

Conceptually:

```text
Client sends:
price = ₹99

Server accepts:
₹99

Transaction total:
₹99
```

This demonstrated insufficient server-side validation of a security-sensitive financial parameter.

---

## Security Impact

The vulnerability could potentially allow an attacker to manipulate transaction prices.

Potential consequences include:

- Financial loss.
- Unauthorized discounts.
- Manipulation of cart totals.
- Abuse of business logic.
- Revenue leakage.
- Integrity issues in order records.
- Potential abuse at scale if no additional controls exist.

The exact business impact depends on the affected endpoint and whether downstream payment systems independently validate the final amount.

---

## Attack Scenario

A malicious user could potentially:

```text
Normal product
      ↓
Observe legitimate transaction request
      ↓
Identify client-controlled price
      ↓
Modify price
      ↓
Submit modified request
      ↓
Backend accepts manipulated amount
      ↓
Reduced transaction value
```

An attacker could potentially automate this behavior if the vulnerable endpoint does not have additional server-side protections.

The attack does not require compromising the application's infrastructure; it exploits the application's trust in client-controlled transaction data.

---

## Remediation

### 1. Never trust client-provided prices

The client should not be the authority for monetary values.

### 2. Resolve prices server-side

Use the product identifier to retrieve the authoritative price.

```text
product_id
    ↓
database/catalog
    ↓
authoritative price
```

### 3. Recalculate totals

The server should calculate:

```text
subtotal
+ applicable taxes
- validated discounts
+ applicable charges
= final amount
```

### 4. Validate discounts server-side

Promotional codes and discounts should be validated against:

- user eligibility,
- product eligibility,
- expiry,
- usage limits,
- maximum discount,
- campaign rules.

### 5. Revalidate before payment

The final payment amount should be independently verified immediately before payment authorization.

### 6. Add anomaly detection

Large deviations between catalog prices and submitted transaction values should generate security/business-logic alerts.

### 7. Add regression tests

Security tests should explicitly attempt:

```text
price = 0
price = negative
price = extremely low
price = extremely high
price = altered decimal value
price = altered currency value
```

using only controlled test environments or authorized accounts.

---

## Disclosure / Acknowledgement Information

**Disclosure type:** Bug bounty / responsible security research

**Researcher:** Manojkumar Gurram

The public version of this writeup intentionally excludes:

- authentication tokens,
- cookies,
- customer information,
- order identifiers,
- internal infrastructure details,
- sensitive production requests.

If the original program provided a disclosure acknowledgement, add it here:

```text
Acknowledgement:
[Insert official acknowledgement, bounty reference, or disclosure date]
```

---

## Lessons Learned

This vulnerability demonstrates an important principle in application security:

> **The client is never an authority for security-sensitive business values.**

Prices, discounts, permissions, account balances, inventory quantities, and transaction totals must be independently validated by the backend.

A visually secure frontend does not protect a transaction if the backend trusts manipulated client input.
