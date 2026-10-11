# 📁 Account Takeover (ATO) — Security Testing Notes

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Test only systems you own or are explicitly authorized to assess. Use test accounts, avoid accessing other users’ data, do not exfiltrate cookies or tokens, and follow the program’s scope and rate limits. Account takeover testing can affect real people; prefer controlled, non-destructive validation.

## 📌 Executive Summary

Account takeover (ATO) occurs when an attacker gains control of another user’s account by abusing weaknesses in authentication, account recovery, session management, or authorization. These notes organize the techniques named in the supplied PDF into a cautious testing workflow focused on verifying controls without taking over real accounts.

## 🔍 Core Testing Methodology

1. **Map authentication and recovery flows**
   - Identify login, sign-up, OAuth, email-change, password-change, password-reset, and OTP endpoints within scope.
   - Use designated test accounts and record the expected workflow.

2. **Review pre-account takeover conditions**
   - In a controlled environment, check how the application handles the same email address across different sign-in methods.
   - Verify that an account is not linked to an identity provider or another account until ownership of the email address is reliably established.
   - Do not attempt to link or access another person’s account.

3. **Assess login controls**
   - Review rate limiting and monitoring for repeated failed login attempts using an approved test account.
   - Check OAuth configuration and JWT validation where those features are present.
   - Verify that OTPs and other authentication tokens are validated server-side, expire appropriately, and cannot be reused.
   - Confirm that malformed input does not bypass authentication.

4. **Assess password-reset controls**
   - Check reset-OTP rate limits using a test account and an agreed request budget.
   - Verify token unpredictability, expiry, single use, and binding to the correct account and purpose.
   - Review whether reset endpoints enforce authorization and validate every required parameter.
   - Check that reset tokens and OTPs are not exposed in URLs, logs, analytics, or response bodies.

5. **Review sensitive-data exposure**
   - Inspect only your own test account’s requests and responses.
   - Look for unnecessary exposure of passwords, OTPs, hashes, session tokens, or reset tokens.
   - Redact secrets from evidence; do not reuse or export tokens belonging to other users.

6. **Review CSRF protections**
   - Check whether email-change and password-change actions require appropriate anti-CSRF protections.
   - Verify that cookie settings and server-side request validation prevent unauthorized cross-site state changes.
   - Do not craft or deliver a request against a real user.

7. **Check IDOR and object-level authorization**
   - With two accounts you control, verify that each account can modify only its own email, password, or reset state.
   - Confirm that changing an identifier does not allow one test account to affect another.
   - Stop if a request would expose or modify data outside the controlled test accounts.

8. **Inspect response and status handling**
   - Compare server responses for valid and invalid test operations.
   - Verify that a success message, redirect, or changed status code does not substitute for server-side authentication or authorization.
   - Confirm the underlying account state using the authorized test account.

9. **Review password/email-change behavior**
   - Confirm that sensitive account changes require appropriate re-authentication or step-up verification.
   - Verify that changing an email address requires proving control of the new address.
   - Check that security notifications and session invalidation behave as intended.

10. **Review token and session security**
    - Check that session identifiers and reset tokens are sufficiently unpredictable, short-lived where appropriate, and invalidated after relevant security changes.
    - Review JWT signature, issuer, audience, expiry, and algorithm validation where JWTs are used.
    - Do not guess or brute-force live tokens.

11. **Review host, CORS, and HTTP parsing configuration**
    - Check whether password-reset links are built from a trusted, configured origin rather than an untrusted Host header.
    - Review CORS policies for exact allowed origins and credential handling.
    - Investigate HTTP request-smuggling risks only in an explicitly authorized environment using an approved test plan; these tests can disrupt services.

12. **Document findings and retest**
    - Record the endpoint, test-account role, expected behavior, observed result, impact, and redacted evidence.
    - Avoid claiming ATO unless control of an account was safely and conclusively demonstrated in a test environment.
    - Retest the fix against the original case and nearby variants.

## 💡 High-Value Bug Bounty Tips & Edge Cases

- **Pre-account takeover:** The source describes a case where an account created with one sign-in method may be incorrectly linked to another method using the same email address. Verify that the application proves email ownership before linking identities.
- **Rate limiting:** The source recommends capturing login requests and reviewing response behavior and length. Use approved, low-volume testing; never run high-volume brute force against production accounts.
- **Sensitive response data:** The PDF mentions OTPs, hashes, and passwords appearing in request or response data. Treat any unexpected exposure as sensitive and redact it immediately.
- **OAuth and JWT configuration:** Test only the authentication flows the application actually uses. Validate identity-provider configuration and token claims rather than assuming every application uses the same setup.
- **OTP and reset-token handling:** Check expiry, one-time use, account binding, rate limits, and server-side validation. Do not publish usable OTPs or reset tokens.
- **CSRF:** The source highlights email- and password-change endpoints. Review whether the server requires a valid anti-CSRF mechanism and appropriate origin protections for cookie-authenticated state changes.
- **IDOR:** Test object ownership with two accounts you control. An identifier change should not allow cross-account changes.
- **Response/status manipulation:** A client-side response alteration is not proof of ATO. The server must independently authorize the operation and persist only valid changes.
- **Weak cryptography:** The source references weak cryptography in password-reset flows. Review token-generation design and cryptographic implementation; do not attempt to recover or misuse real users’ credentials.
- **Email or password-change functions:** Verify ownership checks, re-authentication, confirmation steps, and notifications. Do not substitute a victim’s address into a live request.
- **Sign-up behavior:** Check that third-party identity linking and phone/email verification are explicit and secure. Avoid creating accounts using addresses or phone numbers you do not control.
- **REST tokens:** Verify that tokens are bound to the intended account and action, unpredictable, short-lived where appropriate, and validated server-side. Do not brute-force live reset tokens.
- **Host header injection:** Reset links should use a trusted canonical origin. Do not send deceptive reset links to users.
- **CORS misconfiguration:** A permissive CORS policy can expose data in some credentialed cross-origin scenarios. Verify using controlled accounts and origins; CORS alone does not automatically mean account takeover.
- **Leaked session cookies:** The source references a report involving leaked session cookies. Treat session cookies as credentials; never collect, replay, or publish another user’s cookie.
- **HTTP request smuggling:** The source lists this as an ATO-related technique. It requires careful, specialized testing and may be disruptive; only test with explicit permission and a controlled setup.
- **Origin validation:** The source mentions bypassing digit/origin validation. The exact context is not explained in the PDF, so treat it as a prompt to review origin and input validation—not as a confirmed universal bypass.
- **Reference links:** The PDF includes links to external articles and reports. They are pointers for further study, not proof that the same issue exists in any target.

## 🛠️ Educational Checklists & Safe Examples

The original PDF includes attack-oriented ideas and a JavaScript example that attempts to transmit browser cookies to a server. That exfiltration code is intentionally not reproduced. The examples below use placeholders and benign requests for authorized testing.

### Example: inspect a test-account reset flow

```http
POST /account/password-reset HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "email": "controlled-test-account@example.com",
  "verification_code": "[TEST-OTP]"
}
```

Use a test account and a test OTP issued by your own environment. Never insert a real user’s email address, password, or token.

### Example: safe account-change test record

```text
Target: [Your-Domain]
Endpoint: [In-Scope-Endpoint]
Account: [Controlled-Test-Account]
Action: [Email Change / Password Reset / Login]
Expected result: [Expected secure behavior]
Observed result: [Observed behavior]
Evidence: [Redacted request and response]
Impact: [Demonstrated impact]
```

### ATO review checklist

- [ ] Email ownership is verified before accounts or identity providers are linked.
- [ ] Login attempts and OTP/reset requests are rate-limited.
- [ ] OTPs and reset tokens are unpredictable, purpose-bound, expiring, and single-use.
- [ ] Authentication tokens are never exposed unnecessarily in URLs, logs, or responses.
- [ ] JWTs are validated correctly, including signature and relevant claims.
- [ ] Email/password changes require suitable authorization and verification.
- [ ] CSRF protections cover sensitive cookie-authenticated actions.
- [ ] Object-level authorization prevents cross-account changes.
- [ ] Host headers cannot control password-reset link origins.
- [ ] CORS allows only trusted origins and handles credentials safely.
- [ ] Session identifiers are rotated or invalidated after sensitive changes.
- [ ] Error responses do not reveal secrets or unnecessary account information.
- [ ] Evidence contains no real user credentials, cookies, OTPs, or tokens.

## 🛡️ Mitigation & Remediation

- **Secure account linking:** Require verified email ownership and explicit confirmation before linking accounts or identity providers. Do not rely on an unverified email claim alone.
- **Harden login and OTP flows:** Apply rate limits, abuse monitoring, progressive delays where appropriate, and generic responses that avoid unnecessary account enumeration.
- **Protect password resets:** Generate reset tokens with a cryptographically secure random source, bind them to the correct account and purpose, expire them, enforce single use, and invalidate them after a successful reset.
- **Validate tokens correctly:** For JWTs, verify signatures and enforce expected algorithms, issuer, audience, expiry, and other required claims. Do not trust unvalidated token contents.
- **Prevent sensitive-data leaks:** Never return passwords or password hashes to clients. Keep OTPs, reset tokens, and session secrets out of URLs, analytics, and ordinary logs.
- **Enforce authorization server-side:** Check the authenticated identity and object ownership on every email-change, password-change, and reset operation.
- **Defend against CSRF:** Use anti-CSRF tokens or suitable framework protections, validate origins where appropriate, and configure cookies with suitable `Secure`, `HttpOnly`, and `SameSite` attributes.
- **Protect session lifecycle:** Rotate session identifiers after login and privilege changes. Invalidate relevant sessions after password resets, account recovery, or other sensitive security events.
- **Use trusted reset-link origins:** Construct links from a configured canonical origin rather than untrusted request headers.
- **Restrict CORS:** Allow only trusted origins, avoid broad credentialed CORS policies, and return appropriate CORS headers for each allowed origin.
- **Use safe cryptography:** Use vetted libraries and secure random token generation. Avoid predictable tokens derived from timestamps, sequential IDs, or user information.
- **Harden HTTP infrastructure:** Keep reverse proxies and application servers consistently configured and patched. Assess request-smuggling exposure with a specialized, authorized test plan.
- **Monitor and notify:** Alert on suspicious authentication and recovery patterns, notify users of important account changes, and avoid logging secrets.
- **Regression-test fixes:** Add tests for account linking, recovery, authorization, token reuse, expiry, origin validation, and session invalidation.

## 📚 Source Notes

This guide is based on the three-page *ATO.pdf* supplied for rewriting. The source lists pre-account takeover, rate limiting, sensitive-data exposure, login and password-reset checks, XSS/CSRF/IDOR, response manipulation, weak cryptography, account-change flows, REST tokens, Host header injection, CORS, leaked session cookies, HTTP request smuggling, and origin-validation concerns.

The PDF gives limited detail for several headings and does not establish that any specific target is vulnerable. This guide preserves those topics while replacing unsafe exploit instructions with controlled test guidance and adding general defensive recommendations. The original JavaScript cookie-exfiltration snippet and operational takeover payloads are omitted.

*Educational use only. Test ethically, stay within scope, and disclose findings through the program’s approved process.*
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice end-to-end Account Takeover chaining in authentic environments:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic) | Practitioner | Removing target token parameter to reset administrative password |
| **PortSwigger Academy** | [Basic password reset poisoning](https://portswigger.net/web-security/host-header/lab-host-header-basic-password-reset-poisoning) | Practitioner | Poisoning Host header to intercept reset token via attacker server |
| **PortSwigger Academy** | [Forced OAuth profile linking leading to ATO](https://portswigger.net/web-security/oauth/lab-oauth-forced-oauth-profile-linking) | Practitioner | CSRF in OAuth authorization flow to attach victim account to attacker |
| **PortSwigger Academy** | [OAuth account hijacking via redirect_uri](https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri) | Practitioner | Leaking OAuth authorization codes via open redirect parameter |
| **Hacker101 CTF** | [Micro-CMS v2](https://ctf.hacker101.com/) | Medium | Chaining authentication bypass to achieve complete account takeover |
