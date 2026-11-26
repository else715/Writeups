# BigBasket — Price Tampering

**Researcher:** Manojkumar Gurram  
**Classification:** Business logic flaw in server-side price validation (CWE-602)

## Reported finding

During an authorized assessment of BigBasket’s purchase workflow, the researcher reported that a price supplied by the client influenced the transaction total calculated by the backend. The server should determine the amount payable from its own product catalog, quantity, and applicable pricing rules.

## Potential impact

An incorrect cart total could carry through to an order record. If the payment system also used that total, a customer could be charged less than the intended price. The assessment describes an effect on the transaction calculation. Whether an altered amount reached payment authorization is unconfirmed.

## Remediation

- Look up the current price in the server-side catalog for each product and calculate the subtotal from the validated quantity.
- Apply discounts on the server after checking customer and product eligibility, expiry, and usage limits. Include applicable taxes and charges in the final total.
- Before payment authorization, verify that the amount and currency match the server-calculated order total.
- Add regression tests that assert the order and payment amounts stay consistent with the catalog and eligible discounts throughout checkout. Log discrepancies for investigation.

## Disclosure status

Vendor acknowledgement and remediation status were undocumented in the supplied report.
