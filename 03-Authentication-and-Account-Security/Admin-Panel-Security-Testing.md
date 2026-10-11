# 📁 Admin Panel Security Testing

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Use these checks only on applications you own or are explicitly authorized to test. Keep testing non-destructive, respect program scope and rate limits, and never access, modify, or disclose other users’ data. The source PDF is a short checklist, so some details below are intentionally marked as general testing guidance rather than claims made by the source.

## 📌 Executive Summary

Admin-panel testing evaluates whether an application protects privileged sign-in and administrative functions against weak credentials, flawed input handling, misleading responses, and broken authorization. The goal is to verify the actual security control—not merely whether a page looks restricted or returns a particular HTTP status.

## 🔍 Core Testing Methodology

1. **Confirm scope and identify the login surface**
   - Record the in-scope hostname, login URL, and any linked administrative routes.
   - Use a test account and test data where possible.
   - Avoid guessing against real accounts or running high-volume tests.

2. **Review default-credential exposure**
   - Check documentation and deployment configuration for vendor-documented defaults.
   - Verify only with explicit authorization and a controlled test instance or account.
   - Do not publish working credentials. Record the product/version and remediation instead.

3. **Assess authentication behavior**
   - Test valid and invalid credentials using a designated test account.
   - Compare error messages, redirects, response codes, and response bodies.
   - Check whether failure and success states are determined by the server, rather than trusting a client-side message or status code alone.

4. **Review input handling at the login endpoint**
   - Map the username, password, and other parameters.
   - Test how the application handles unexpected types, empty values, encoding, and malformed input.
   - Use harmless test values and observe whether errors expose implementation details.
   - Do not use injection attempts to access real accounts or data.

5. **Check request parsing and content types**
   - Compare the documented request format with the format actually accepted by the endpoint.
   - In a controlled environment, check whether JSON, form-encoded data, and unexpected data types are validated consistently.
   - Verify that unsupported methods and content types are rejected safely.

6. **Inspect response handling**
   - Observe HTTP status codes, redirects, response bodies, and application-level success/failure indicators.
   - Determine whether the server independently verifies authentication and authorization.
   - Treat a changed status code or client-visible success message as a lead—not proof of a bypass.

7. **Review rate limiting and brute-force protections**
   - Confirm that repeated failed attempts trigger appropriate throttling, progressive delays, or other risk controls.
   - Use a small, agreed test window and test account.
   - Do not run credential-stuffing or high-volume password-guessing activity.

8. **Discover exposed administrative routes carefully**
   - Review robots files, application links, public documentation, and in-scope route lists.
   - If directory/content discovery is permitted, use an approved wordlist with conservative request rates.
   - Do not treat the existence of a route as a vulnerability; verify its access controls.

9. **Check for information leaks in pages and responses**
   - Review page source, comments, JavaScript, error messages, and login responses for secrets or sensitive implementation details.
   - Never collect or publish real passwords, session tokens, API keys, or personal data. Redact evidence.

10. **Test authorization after login**
    - Use separate low-privilege and administrator test accounts where available.
    - Request administrative pages and actions directly while authenticated as the lower-privilege user.
    - Verify access on the server for every sensitive action and object, not just in the interface.

11. **Check technology-specific input risks**
    - Where the application uses SQL, NoSQL, XPath, or LDAP queries, review whether untrusted input is safely parameterized or escaped for the relevant query language.
    - Use approved, non-destructive test cases in a lab or authorized environment.
    - Stop if a test risks changing data, enumerating users, or exposing records.

12. **Document and retest**
    - Record the affected endpoint, account role, request/response evidence, impact, and exact reproduction conditions.
    - Redact secrets and unrelated user data.
    - Retest after the fix and confirm that both direct requests and alternate request formats are protected.

## 💡 High-Value Bug Bounty Tips & Bypasses

- **Default credentials:** The source lists common default-credential patterns. Treat these as a reminder to check vendor setup guidance—not as a list to try against arbitrary systems.
- **Error-based and timing behavior:** The source calls out error-based and time-based SQL injection testing. Unexpected errors or consistent timing differences may indicate unsafe query handling, but require careful validation in an authorized test environment.
- **Encoding variations:** The source mentions URL encoding and Base64 encoding in the context of XSS tests. Encoding is not sanitization; the application must validate and contextually encode input and output. Do not assume that encoding a string proves a vulnerability or bypasses a filter.
- **Response manipulation:** The source suggests comparing status codes such as `200`, `302`, and `403`, and checking application-level success/failure values. A client-side response change does not grant server-side privileges; verify access to a protected resource with an authorized test account.
- **Brute-force protection:** Review whether rate limits apply consistently across relevant login routes and account identifiers. Do not exceed program limits or use real users’ accounts.
- **Directory discovery:** The source references a wordlist for fuzzing. Use only in-scope targets, approved wordlists, and conservative rates. A discovered path is not itself proof of unauthorized access.
- **Removing request parameters:** The source describes removing a password parameter from a request and forwarding it. This can help assess server-side validation in a controlled test, but should never be used to impersonate another user or gain unauthorized access. Confirm that required credentials are validated server-side.
- **Page source and comments:** Login pages and HTML comments can accidentally reveal sensitive paths or configuration. Secrets should never be embedded in client-delivered content.
- **Type confusion and JSON parsing:** The source recommends checking JSON bodies, including boolean values, and testing parser errors. APIs should enforce schemas and reject unexpected types consistently.
- **NoSQL query construction:** The source flags NoSQL injection. Use safe lab cases to confirm whether input is treated as data rather than executable query structure.
- **XPath and LDAP inputs:** These are relevant only where the application builds XPath or LDAP expressions. Validation and escaping must match the specific language and context.
- **Authorization is separate from authentication:** Successfully signing in does not prove that the user may access every admin function. Test role and object-level permissions independently.
- **PHP comparison errors:** The source ends with a PHP comparison-error heading but provides no further explanation. Do not infer a specific weakness without evidence from the application.

## 🛠️ Educational Checklists & Payloads

The source PDF includes credential examples and injection-oriented strings. To keep this public guide safe and reusable, the examples below are placeholders and benign validation cases—not authentication-bypass payloads.

### Safe request-validation examples

Use these only against a local lab or an explicitly authorized test endpoint.

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=test-user&password=invalid-test-value
```

```http
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "username": "test-user",
  "password": "invalid-test-value"
}
```

### Test checklist

- [ ] The application does not rely on undocumented or unchanged default credentials.
- [ ] Failed authentication never creates an authenticated session.
- [ ] Server-side authentication is required for every protected route.
- [ ] Authorization is checked for every privileged action and sensitive object.
- [ ] Unexpected parameter types and malformed requests are rejected safely.
- [ ] Database and directory queries use safe parameterization or context-appropriate escaping.
- [ ] Errors do not expose query text, stack traces, secrets, or internal configuration.
- [ ] Rate limits and lockout/risk controls behave as documented.
- [ ] Redirects and HTTP status codes cannot substitute for an authorization decision.
- [ ] HTML output is contextually encoded; input validation is not mistaken for output encoding.
- [ ] Request content types and HTTP methods are validated consistently.
- [ ] Evidence is redacted and contains no real credentials, tokens, or unrelated user data.

### Evidence template

```text
Target: [Your-Domain]
Endpoint: [In-Scope-Path]
Test account/role: [Authorized-Test-Account]
Issue category: [Authentication / Injection / Authorization / Information Disclosure]
Expected behavior: [Expected secure behavior]
Observed behavior: [Observed result]
Impact: [Demonstrated impact, without speculation]
Evidence: [Redacted request/response]
Reproduction notes: [Minimal, non-destructive steps]
```

## 🛡️ Mitigation & Remediation

- **Remove default credentials:** Require unique credentials during setup, enforce a secure initial-password flow, and make default accounts inactive or force credential changes before use.
- **Use parameterized queries:** Avoid building SQL, NoSQL, XPath, or LDAP expressions by concatenating untrusted input. Use the relevant library’s safe parameterization APIs and context-specific escaping where applicable.
- **Validate input types and schemas:** Enforce required fields, allowed types, lengths, and formats on the server. Reject unexpected JSON types and unsupported request formats.
- **Encode output correctly:** Apply context-aware output encoding to prevent XSS. Do not rely on URL encoding or Base64 encoding as a security control.
- **Enforce server-side authorization:** Check the authenticated identity and required role/permission on every protected request. Never rely only on hidden buttons, client-side state, status codes, or redirect behavior.
- **Harden authentication:** Apply rate limiting, monitoring, secure session handling, generic error messages where appropriate, and multifactor authentication for privileged accounts.
- **Protect sensitive routes:** Remove unnecessary admin interfaces from public exposure where feasible, but still enforce authentication and authorization. Do not rely on obscurity or a non-obvious URL.
- **Handle errors safely:** Return generic client-facing errors and keep diagnostic details in access-controlled logs. Never expose credentials, tokens, query fragments, or stack traces.
- **Secure the session lifecycle:** Rotate session identifiers after authentication and privilege changes; use appropriate `Secure`, `HttpOnly`, and `SameSite` cookie attributes; invalidate sessions on logout and relevant account/security events.
- **Monitor and retest:** Log authentication failures and privileged actions without logging passwords or session secrets. Add regression tests for the reported issue and related request variations.

## 📚 Source Notes

This guide is based on the two-page *Admin panel.pdf* supplied for this rewrite. The PDF is a compact checklist and does not provide complete explanations for every heading. Where the source only names a technique, this document presents it as a testing lead and adds general defensive guidance; it does not claim that a particular target is vulnerable.

*Educational use only. Test ethically, stay within scope, and disclose findings through the program’s approved process.*
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice administrative panel discovery, authorization bypasses, and access control:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Unprotected admin functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality) | Apprentice | Locating hidden administrative panel path via robots.txt |
| **PortSwigger Academy** | [Unprotected admin functionality with unpredictable URL](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url) | Apprentice | Extracting random admin URL from client-side JavaScript source |
| **PortSwigger Academy** | [URL-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented) | Practitioner | Bypassing admin path restrictions via `X-Original-URL: /admin` |
| **Hack The Box** | [HTB Machine: BoardLight](https://app.hackthebox.com/machines/BoardLight) | Easy | Default credential takeover of internal Dolibarr CRM admin console |
| **TryHackMe** | [Admin Panel Discovery Room](https://tryhackme.com/room/adminpanel) | Easy | Wordlist fuzzing, virtual host enumeration, and header bypasses |
