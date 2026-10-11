# 📁 Authentication Bypass — Security Testing Notes

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Test only applications you own or are explicitly authorized to assess. Use controlled test accounts, respect scope and rate limits, and avoid actions that could lock out users, change another person’s account, or expose private data.

## 📌 Executive Summary

Authentication bypass weaknesses occur when an application grants access or performs a sensitive action without correctly verifying identity, session state, or required security checks. This guide organizes the two-page source checklist into a safe workflow for evaluating authentication controls, account recovery, registration, and session management.

## 🔍 Core Testing Methodology

1. **Review protected POST endpoints**
   - Identify endpoints that should require an authenticated session.
   - Using a controlled test account, check whether a direct request without a valid session is rejected.
   - Verify that a URL or endpoint is not considered secure merely because it is difficult to guess.

2. **Check CAPTCHA and proxy-header trust**
   - Review whether CAPTCHA validation is performed server-side and tied to the relevant action.
   - Verify that client-supplied forwarding headers are not treated as proof of identity or used to bypass security controls.
   - Use a local test environment; do not automate CAPTCHA evasion against production systems.

3. **Review password-confirmation requirements**
   - Determine whether sensitive actions—such as changing an email address, changing a password, deleting an account, or managing two-factor authentication—require recent authentication or step-up verification.
   - Verify the rule using test accounts and document the expected behavior.

4. **Check registration email verification**
   - Confirm that the application verifies control of an email address before treating it as verified.
   - Check whether unverified accounts receive access to features that require a verified identity.

5. **Assess form rate limiting**
   - Review rate limits for forms using an approved, small test budget.
   - Confirm that repeated requests are throttled or otherwise controlled.
   - Do not replay requests at scale or cause service degradation.

6. **Assess login throttling and CAPTCHA**
   - Use a test account to review behavior after a small number of failed login attempts.
   - Confirm that controls are enforced server-side and consistently across relevant login routes.
   - Avoid brute-force campaigns and stop at the agreed testing threshold.

7. **Review username/email enumeration**
   - Compare responses for controlled existing and non-existing test identifiers.
   - Look for differences in status code, error text, response length, timing, or proxy-related behavior.
   - Do not test against real users’ identifiers or publish account-existence information.

8. **Review password policy**
   - Check whether the application enforces documented password length and complexity requirements.
   - Test policy validation only on your own test account.
   - Consider whether weak or commonly used passwords are rejected and whether password changes require appropriate verification.

9. **Verify transport security**
   - Inspect registration, login, and account-recovery requests in a controlled environment.
   - Confirm that credentials and tokens are sent only over HTTPS and that HTTP is redirected or rejected safely.
   - Use browser developer tools or an authorized proxy to inspect the connection; avoid capturing other users’ traffic.

10. **Review authentication responses**
    - Compare status codes, error messages, and response behavior for test cases.
    - Ensure these differences do not disclose sensitive account information or replace server-side authentication checks.

11. **Test password-reset token binding**
    - Use two test accounts you control.
    - Request a reset for one account and verify that the resulting token can only reset that account’s password.
    - Confirm that tokens expire, are single-use, and cannot be substituted across accounts or sessions.

12. **Review session and recovery interactions**
    - Check whether logging out, changing a password, or completing recovery invalidates sessions and outstanding recovery tokens as intended.
    - Confirm that recovery actions are bound to the correct account and that account identifiers cannot be swapped to affect another account.
    - Do not use a reset link or token belonging to another person.

13. **Record findings and retest**
    - Capture the endpoint, account role, expected behavior, observed behavior, and redacted evidence.
    - Demonstrate impact only with controlled accounts.
    - Retest after remediation and check related flows for regressions.

## 💡 High-Value Bug Bounty Tips & Edge Cases

- **Direct endpoint access:** The source recommends checking whether POST authentication URLs can be accessed without a session. A reachable endpoint is not automatically vulnerable; verify whether protected data or actions are actually available.
- **Guessable URLs:** If a sensitive endpoint uses predictable paths, assess whether access controls are still enforced. URL secrecy is not a substitute for authentication.
- **Forwarding headers:** The source mentions `X-Forwarded-For` in connection with CAPTCHA bypass. Applications must not trust arbitrary client-supplied forwarding headers for identity, authorization, or security decisions.
- **Password confirmation:** Pay particular attention to email changes, password changes, account deletion, and 2FA management. Sensitive changes may require re-authentication or step-up verification.
- **Email verification:** Registration should not mark an email as verified until the user proves control of it.
- **Rate limiting:** The source proposes repeating a form request and observing restrictions. Use only a small approved test sample; do not send dozens of requests to a production service without explicit permission.
- **Login protections:** Review lockout, throttling, and CAPTCHA behavior with a test account. Do not run brute-force attempts against real accounts.
- **Account enumeration:** Compare status codes, errors, response times, and response lengths for controlled test identities. Avoid exposing or collecting information about real users.
- **Password policy:** A policy that accepts only numeric, lowercase, common, or very short passwords may be inadequate. Validate policy requirements without changing another user’s credentials.
- **HTTP registration:** The source flags registration over HTTP. Credentials and verification tokens must be protected in transit with HTTPS.
- **Transport inspection:** A proxy can help inspect your own test traffic; packet capture should be limited to traffic you are authorized to observe.
- **Reset-token binding:** The source describes an issue where a reset code may be used in an unexpected account flow. Verify that every reset token is bound to the intended account, purpose, and expiry.
- **Session management:** Password reset, password change, logout, and account recovery should have clearly defined session-invalidation behavior.
- **Source limitation:** The PDF’s two final examples describe recovery/session interactions but do not fully explain the underlying application logic. Treat them as prompts for controlled testing, not proof of a universal exploit.

## 🛠️ Educational Checklists & Safe Examples

The source is a compact checklist and does not provide a full application, API schema, or reproducible lab. The following examples are generic and intended for an authorized test environment.

### Example: verify a protected endpoint rejects an unauthenticated request

```http
POST /account/settings HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "action": "view-test-settings"
}
```

**Expected secure result:** The server rejects the request when a valid authenticated session is absent. Use a non-sensitive test endpoint and do not include real credentials or session cookies in public notes.

### Example: compare safe authentication outcomes

```text
Test identity: [Controlled-Test-Account]
Request: [In-Scope-Login-or-Recovery-Endpoint]
Case A: [Valid test identifier / invalid test secret]
Case B: [Non-existent synthetic test identifier]
Record: [Status, generic error, response length, timing]
Sensitive data: [Redacted]
```

### Authentication-bypass checklist

- [ ] Protected endpoints reject requests without a valid session.
- [ ] Access controls are enforced server-side, regardless of URL predictability.
- [ ] CAPTCHA results are verified server-side and bound to the action.
- [ ] Forwarding headers cannot bypass authentication or rate limits.
- [ ] Sensitive account changes require suitable re-authentication.
- [ ] Email ownership is verified before an address is trusted.
- [ ] Forms and login endpoints enforce appropriate rate limits.
- [ ] Authentication responses minimize account enumeration.
- [ ] Password policy requirements are enforced on the server.
- [ ] Registration, login, and recovery use HTTPS.
- [ ] Password-reset tokens are unpredictable, expiring, single-use, and account-bound.
- [ ] Password changes and recovery invalidate relevant sessions/tokens.
- [ ] Evidence is redacted and uses only controlled test accounts.

## 🛡️ Mitigation & Remediation

- **Enforce authentication centrally:** Apply server-side authentication middleware to every protected endpoint, including POST routes and APIs. Do not rely on hidden UI controls or obscure URLs.
- **Validate CAPTCHA correctly:** Verify CAPTCHA tokens on the server, check their purpose and expiry, and bind them to the relevant action. Do not use untrusted request headers as a substitute.
- **Require step-up verification:** Ask users to re-authenticate or use a stronger verification step before high-risk account changes.
- **Verify email ownership:** Use time-limited, single-use verification links or codes and do not mark addresses as verified before successful confirmation.
- **Apply abuse controls:** Rate-limit login, recovery, registration, and other sensitive forms. Monitor suspicious patterns and use proportionate progressive controls.
- **Reduce enumeration:** Use consistent public-facing messages and carefully designed response behavior for existing and non-existing accounts.
- **Enforce robust password policy:** Set appropriate minimum length, reject known-compromised passwords where feasible, and support password managers. Avoid unnecessarily restrictive character rules.
- **Require HTTPS:** Redirect or reject insecure HTTP access and enable HSTS where appropriate. Protect credentials and tokens in transit.
- **Use secure recovery tokens:** Generate tokens with a cryptographically secure random source, bind them to one account and one purpose, expire them, and invalidate them after use or a new recovery request.
- **Harden session lifecycle:** Rotate session IDs after authentication and privilege changes; invalidate sessions after password resets or other security events according to the application’s risk model.
- **Protect sensitive actions against CSRF:** Apply suitable framework protections, validate requests, and configure cookies with appropriate `Secure`, `HttpOnly`, and `SameSite` attributes.
- **Audit and regression-test:** Log relevant security events without logging passwords or tokens. Add tests for unauthenticated access, account binding, rate limits, and session invalidation.

## 📚 Source Notes

This guide is based on the supplied two-page *auth bypass.pdf*. Page 1 covers protected POST endpoints, CAPTCHA and `X-Forwarded-For`, password confirmation, email verification, form/login rate limits, username/email enumeration, password policy, and registration over HTTP. Page 2 covers secure transport, enumeration signals, and password-reset/session-management interactions.

The PDF provides checklist-style prompts rather than complete technical explanations. This rewrite preserves those topics, uses safe test examples, and adds general defensive guidance. It does not claim that any particular application is vulnerable.

*Educational use only. Test ethically, stay within scope, and disclose findings through the program’s approved process.*
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice authentication logic bypasses, token forgery, and signature tampering:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [JWT authentication bypass via unverified signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature) | Apprentice | Tampering with JWT payload without updating the signature |
| **PortSwigger Academy** | [JWT authentication bypass via flawed signature verification](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification) | Practitioner | Modifying algorithm to `none` to bypass JWT verification |
| **PortSwigger Academy** | [JWT authentication bypass via weak HMAC secret key](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-hmac-secret-key) | Practitioner | Cracking weak JWT signing secrets using Hashcat |
| **TryHackMe** | [Authentication Bypass Room](https://tryhackme.com/room/authenticationbypass) | Easy | Username enumeration, cookie tampering, and logic flaws |
| **Hack The Box** | [HTB Machine: Shoppy](https://app.hackthebox.com/machines/Shoppy) | Easy | Bypassing authentication using NoSQL object injection (`admin' || '1'=='1`) |
