# 📁 Business Logic Vulnerabilities — Security Testing Notes

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Test only applications you own or are explicitly authorized to assess. Use dedicated test accounts, synthetic records, and reversible actions. Do not alter real prices, payments, exam results, other users’ data, or production entitlements.

## 📌 Executive Summary

Business logic vulnerabilities occur when an application’s workflow or rules can be used in an unintended way, even if individual endpoints appear to work correctly. The supplied three-page PDF covers pricing, carts, reviews, coupons, fees, currency conversion, premium features, refunds, comments, parameter manipulation, privilege escalation, file access, and other workflow controls.

## 🔍 Core Testing Methodology

1. **Map the intended workflow**
   - Document each step, required state, role, price, limit, and business rule.
   - Use the application normally with a dedicated test account and record baseline requests and responses.

2. **Identify business-critical parameters**
   - Review requests for price, quantity, discount, delivery fee, currency, subscription state, role, account ID, and workflow status.
   - Determine which values should be calculated or validated by the server.

3. **Test pricing and quantity rules**
   - In a test environment, check that the server recalculates totals and rejects invalid quantities or client-supplied prices.
   - Verify negative, zero, excessive, or inconsistent values are rejected where the business rules do not permit them.

4. **Review profile and object access**
   - Use two accounts you control to verify that changing an identifier cannot expose another account’s profile or content.
   - Confirm ownership and authorization checks apply to every request.

5. **Check shopping-cart and checkout state**
   - Verify that the server confirms cart ownership, item availability, quantities, and payment status.
   - Ensure an order cannot be marked purchased without the required payment and fulfillment workflow.

6. **Review review/rating functionality**
   - Check whether verified-purchase or verified-review labels are assigned by the server.
   - Test whether duplicate reviews or invalid ratings are accepted, using test products and accounts.

7. **Review coupon and discount rules**
   - Check expiry, eligibility, usage limits, account binding, and whether a code can be reused or applied concurrently.
   - Confirm the server calculates discounts and validates which products qualify.

8. **Review fees and currency conversion**
   - Verify delivery fees and currency conversions are calculated from trusted server-side rules.
   - Confirm the final amount cannot be reduced by changing client-side parameters.

9. **Review premium-feature entitlements**
   - Check that premium access is determined by a trusted server-side subscription or entitlement record.
   - Verify that changing local storage, cookies, or client-visible flags does not grant premium functionality.

10. **Review refunds and cancellations**
    - Check that refund eligibility, amounts, and cancellation state are validated server-side.
    - Use test transactions; do not create repeated refunds or manipulate real financial transactions.

11. **Review comments and user-generated content**
    - Verify per-thread and per-user limits, ownership, moderation, and impersonation protections.
    - Test only with synthetic accounts and content.

12. **Review parameter and response handling**
    - Check whether unexpected fields or client-modified response values influence server-side decisions.
    - Confirm that payment status, 2FA state, roles, grades, and other critical values come from trusted server-side state.

13. **Review workflow transitions**
    - Check whether steps can be skipped, repeated, or performed out of order, including redirects and payment transfers.
    - Verify each transition checks the current state and caller permissions.

14. **Review cookies, tokens, and access control**
    - Determine whether cookies or tokens carry security-sensitive state and whether the server validates them appropriately.
    - Never forge or replay another user’s session; use controlled test accounts.

15. **Review infrastructure-backed features**
    - Where LDAP or other directory services support application decisions, verify inputs are safely handled and authorization remains enforced.
    - Use only approved, non-destructive test cases.

16. **Review file access and exports**
    - Verify export endpoints enforce object ownership and authorization.
    - Use synthetic files and confirm that another test account cannot retrieve them.

17. **Document and retest**
    - Record the intended rule, controlled test case, observed behavior, impact, and redacted evidence.
    - Retest the original flow and related state transitions after remediation.

## 💡 High-Value Bug Bounty Tips & Edge Cases

The PDF lists the following areas. These are test ideas from the source, not evidence that any target is vulnerable.

- **Price manipulation:** Check whether prices are recalculated by the server and whether invalid values are rejected.
- **Profile retrieval:** The source illustrates changing a profile identifier. Verify object-level authorization with accounts you control.
- **Shopping-cart logic:** Confirm that checkout validates cart ownership, item state, quantity, and payment before granting purchased access.
- **Review functionality:** Check verified-purchase labels, rating boundaries, duplicate reviews, file-upload protections, and CSRF protection where relevant.
- **Coupon reuse:** Verify single-use and concurrency rules, account eligibility, expiry, and product restrictions.
- **Delivery-charge abuse:** Confirm shipping fees are calculated server-side and cannot be reduced by tampering with request parameters.
- **Currency conversion:** Verify conversion rates and refund calculations are authoritative and consistent. Do not use real transactions to test arbitrage.
- **Premium features:** Confirm access is controlled by server-side entitlements rather than local storage or client-visible flags.
- **Refund abuse:** Test cancellation and refund state transitions with sandbox transactions; verify duplicate or concurrent requests cannot produce multiple refunds.
- **Cart/wishlist abuse:** Validate quantities, stock, ownership, and movement between wishlist and cart. Reject invalid quantities according to the application’s rules.
- **Thread comments:** Check comment limits, ownership, and impersonation protections.
- **Parameter tampering:** The source highlights payment and other critical fields, extra parameters, and response manipulation. Security-critical decisions must be based on trusted server state.
- **Product-price manipulation:** The source links to an external video, but provides no details of its contents. No further claim is made here.
- **Exam-result manipulation:** The PDF describes a workflow involving exam answers, retakes, and certificate issuance, with a JSON response field used in the example. Treat grades and certificates as high-integrity records; test only with authorized sandbox data.
- **Authentication flags and privilege escalation:** Verify role and privilege changes are authorized server-side and cannot be controlled by ordinary users.
- **Critical parameter manipulation and data disclosure:** Check whether changing request parameters can expose profiles or content outside the caller’s permissions.
- **Developer-cookie and business-logic bypass:** Confirm that client cookies are not trusted as the sole source of authorization or entitlement state.
- **LDAP parameter handling:** Where LDAP is used, validate and safely encode inputs for the relevant query context, and maintain application-level authorization.
- **Business-constraint exploitation:** Test limits, transfer amounts, and other hidden business rules against documented constraints in a test environment.
- **Business-flow bypass:** Confirm payment, redirect, and transfer workflows cannot skip required verification or payment steps.
- **Identity/profile extraction:** Ensure profile IDs and session tokens cannot be guessed or used to access another user’s information.
- **File or unauthorized URL access:** Verify that exported files and internal resources require authorization; do not fetch other users’ files.
- **Null payloads and password-change flows:** The source ends with these headings but provides little detail. Treat them as prompts to review input validation and current-password requirements, not as confirmed weaknesses.

## 🛠️ Educational Checklists & Safe Examples

### Example: safe cart validation

```http
POST /api/cart/items HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer [TEST-ACCOUNT-TOKEN]

{
  "product_id": "[TEST-PRODUCT-ID]",
  "quantity": 1
}
```

**Expected secure behavior:** The server checks product availability, quantity limits, cart ownership, and authoritative pricing. The client must not be able to set the final price or mark the order as paid.

### Example: business-rule test record

```text
Target: [Your-Domain]
Workflow: [Cart / Coupon / Refund / Subscription / Export]
Test account: [Controlled-Test-Account]
Business rule: [Expected rule or limit]
Input variation: [Synthetic test case]
Expected result: [Rejected or safely constrained]
Observed result: [Redacted outcome]
Impact: [Demonstrated impact]
Evidence: [Redacted request/response]
```

### Business logic checklist

- [ ] Prices, totals, fees, taxes, and discounts are calculated or validated server-side.
- [ ] Quantity, stock, and purchase limits are enforced.
- [ ] Checkout requires the correct payment and order state.
- [ ] Reviews and verified-purchase indicators cannot be forged by clients.
- [ ] Coupons enforce expiry, eligibility, reuse, and concurrency limits.
- [ ] Refunds and cancellations are idempotent and correctly authorized.
- [ ] Premium access derives from server-side entitlements.
- [ ] Sensitive account changes require appropriate verification.
- [ ] User, group, and object ownership is checked on every request.
- [ ] Grades, certificates, payment state, and privilege flags cannot be client-controlled.
- [ ] Cookies and tokens are validated and are not the sole trusted source of critical state.
- [ ] File exports and internal URLs enforce authorization.
- [ ] Errors and responses do not disclose sensitive data.
- [ ] Tests use synthetic accounts and sandbox transactions.

## 🛡️ Mitigation & Remediation

- **Keep business rules on the server:** Treat all client-supplied values as untrusted. Recalculate prices, totals, fees, discounts, and eligibility on the server.
- **Enforce authorization for every object and action:** Check caller identity, ownership, role, and workflow state before reading or changing records.
- **Model workflows as explicit state machines:** Permit only valid transitions. Require payment confirmation before fulfillment and enforce prerequisites for refunds, retakes, certificates, and account changes.
- **Make sensitive operations idempotent:** Use unique transaction identifiers and concurrency controls to prevent duplicate refunds, coupon redemption, or order processing.
- **Validate limits and invariants:** Enforce quantity bounds, stock availability, rating ranges, coupon eligibility, usage limits, and account-specific constraints.
- **Protect premium entitlements:** Derive subscription status and feature access from trusted server-side records, not local storage or untrusted cookies.
- **Protect session state:** Use secure session management and validate tokens. Never accept client-controlled role or privilege flags as authoritative.
- **Secure user-generated content and uploads:** Validate file types and sizes, enforce ownership, apply output encoding, and protect state-changing actions against CSRF where applicable.
- **Secure integrations:** Parameterize or safely encode inputs for LDAP and other query languages, and enforce authorization independently of the integration.
- **Protect exports and files:** Check permissions on each download and export request; use short-lived access links when appropriate.
- **Log and alert on anomalies:** Monitor unusual price changes, repeated refunds, coupon reuse, privilege changes, and invalid workflow transitions without logging secrets.
- **Regression-test business rules:** Cover boundary values, duplicate/concurrent requests, role changes, and out-of-order workflow steps.

## 📚 Source Notes

This guide is based on the supplied three-page *bussiness logic.pdf* (filename spelling preserved). Page 1 covers price manipulation, profile retrieval, shopping carts, reviews, coupons, delivery fees, currency conversion, premium features, refunds, cart/wishlist behavior, comments, and parameter tampering. Page 2 covers further parameter manipulation, product-price and exam-result manipulation, privilege escalation, and critical parameter access. Page 3 covers LDAP parameters, business constraints and flow bypasses, identity/profile extraction, file access, null payloads, and password-change checks.

The source is a checklist and does not provide complete reproduction steps or verified outcomes for every topic. The guide preserves the source’s subject areas, uses safe placeholder examples, and adds general defensive recommendations without claiming that a particular application is vulnerable.

*Educational use only. Test ethically, stay within scope, and disclose findings through the program’s approved process.*
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice finding complex application logic and state transition vulnerabilities:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Excessive trust in client-side controls](https://portswigger.net/web-security/logic-flaws/lab-logic-flaws-excessive-trust-in-client-side-controls) | Apprentice | Cart price tampering directly in intercepted POST request |
| **PortSwigger Academy** | [High-level logic vulnerability](https://portswigger.net/web-security/logic-flaws/lab-logic-flaws-high-level-logic-vulnerability) | Practitioner | Negative quantity tampering leading to zero-dollar checkout |
| **PortSwigger Academy** | [Inconsistent handling of exceptional input](https://portswigger.net/web-security/logic-flaws/lab-logic-flaws-inconsistent-handling-of-exceptional-input) | Practitioner | Email truncation trick to bypass administrative domain checks |
| **PortSwigger Academy** | [Flawed enforcement of business rules](https://portswigger.net/web-security/logic-flaws/lab-logic-flaws-flawed-enforcement-of-business-rules) | Practitioner | Infinite discount coupon stacking logic flaw |
| **Hacker101 CTF** | [Petshop Pro](https://ctf.hacker101.com/) | Medium | Cart price tampering, promo code reuse, balance manipulation |
| **TryHackMe** | [Business Logic Flaws Room](https://tryhackme.com/room/businesslogicflaws) | Easy | Practical checkout, state transition, and race condition flaws |
