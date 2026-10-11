# 📁 Password Reset & Account Recovery — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Password reset flows are the primary target for Account Takeover (ATO) attacks. Test strictly against assets covered by your authorized bug bounty scope. **Never trigger password reset emails against third-party users or attempt to hijack accounts you do not own.** Use two dedicated, controlled test accounts and private test inboxes.

---

## 📌 Executive Summary

The **Password Reset / Account Recovery flow** is among the most sensitive mechanisms in web applications. Because password reset allows users who have lost their credentials to regain access, any architectural or implementation flaw in this workflow can grant an attacker unauthorized access to any user account.

Common flaws include **token entropy deficits**, **Host header poisoning**, **token leakage via HTTP Referer or analytics scripts**, **token reuse**, and **failure to invalidate existing active sessions upon password change**.

```
+---------------------------------------------------------------------------------+
|                        Password Reset Attack Surface                            |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  Token Leakage Vectors            Host Header Poisoning            Token Validation Flaws
  - Referer header to CDNs         - Reset link points to evil.com  - Predictable / Non-expiring
  - Leaked in API response body    - Token harvested in logs        - Cross-account token replay
```

---

## 🔍 Core Discovery & Testing Methodology

### 1. Token Generation & Entropy Analysis
Password reset tokens must be cryptographically secure and unpredictable:
- **Entropy Check:** Is the token a high-entropy string (e.g. 256-bit random UUID or cryptographically random hex/base64 string)?
- **Predictability & Sequence:** If you request 5 consecutive password resets, are the tokens sequential, timestamp-based, or derived from user attributes (MD5 of email + timestamp)?
- **Short Numeric Codes (OTP):** If the reset uses a 4-digit or 6-digit PIN instead of a URL token, verify whether strict rate limiting and brute-force lockouts are enforced.

---

### 2. Password Reset Poisoning via Host Header
When an application constructs the reset URL in the email body dynamically using the incoming HTTP `Host` header:
1. Intercept the password reset request for your controlled test account.
2. Modify the `Host` or `X-Forwarded-Host` header to point to your controlled domain (e.g. `canary.example.com`).
3. Send the request and check the resulting email in your inbox.
4. **Vulnerable Indicator:** The email link points to `https://canary.example.com/reset?token=XYZ`, leaking the secret token to the external host if clicked.
5. *See dedicated guide:* [`Host-Header-Injection-Security-Testing.md`](../02-Web-Vulnerabilities/Host-Header-Injection-Security-Testing.md).

---

### 3. Token Leakage via HTTP Referer Header
When a user clicks the password reset link in their email, they navigate to:
`https://example.com/reset-password?token=SECRET_TOKEN_123`
If this password reset webpage loads external third-party assets (such as Google Fonts, CDN scripts, YouTube embeds, or analytics pixels like Google Analytics / Facebook Pixel):
1. The browser sends an HTTP `Referer` header to the third-party domain containing the full URL:
   `Referer: https://example.com/reset-password?token=SECRET_TOKEN_123`
2. The third-party analytics provider or CDN now possesses the secret reset token in its access logs!
3. **Verification:** Inspect network requests on the reset page in DevTools. Check if any outgoing third-party request includes the token in the `Referer` header.

---

### 4. Token Invalidation & Lifecycle Flaws
Test the token state transitions:

| Test Case | Procedure | Expected Secure Behavior | Vulnerable Indicator |
| :--- | :--- | :--- | :--- |
| **Single-Use Enforcement** | Complete password reset once, then attempt to reuse the exact same link a second time. | Token is rejected (`400 Bad Request` / Expired). | Token is accepted again, allowing repeat resets! |
| **Time-to-Live (TTL)** | Request a reset link and wait for the documented expiry window (e.g. 15–30 minutes). | Token expires and is rejected. | Token remains valid days or weeks later. |
| **Invalidation on New Request**| Request Reset Link 1, then immediately request Reset Link 2. Test Link 1. | Link 1 is superseded and invalidated by Link 2. | Both Link 1 and Link 2 remain valid simultaneously. |
| **Token Bound to Account** | Generate a token for Account A. Attempt to submit it with Account B's email. | Rejected: Token bound strictly to Account A. | Token resets Account B's password! |

---

### 5. Existing Session Invalidation
When a user completes a password reset (or changes their password), all other active sessions and cookies across all devices must be invalidated:
1. Log into your test account on Browser 1.
2. Open Browser 2 and execute a password reset.
3. Return to Browser 1 and refresh a protected page (`GET /api/user/profile`).
4. **Vulnerable Indicator:** Browser 1 remains logged in. If an account was compromised, the legitimate owner resetting their password does not evict the attacker!

---

### 6. User Enumeration via Reset Responses
Compare server responses when requesting password resets for:
- Registered email: `testuser@example.com`
- Unregistered email: `nonexistent9921@example.com`
- *Vulnerable Indicator:* Response status codes, JSON error messages, or response times differ (e.g. `User not found` vs `Email sent`), disclosing whether an account exists.

---

## 🛠️ Safe Educational Verification Template

### Safe HTTP Request Example
```http
POST /api/v1/auth/reset-password HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "email": "controlled-tester@example.com",
  "token": "a98f12c-test-token-value",
  "new_password": "NewSecurePassword123!"
}
```

### Response Validation
Confirm that the password is changed, the token cannot be reused, and pre-existing session cookies on other browser profiles are rejected with `401 Unauthorized`.

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Cryptographically Secure Tokens Bound to User State
Generate high-entropy tokens using secure random number generators (`crypto.randomBytes(32)` in Node.js, `secrets.token_urlsafe(32)` in Python). Store a cryptographic hash (SHA-256) of the token in the database, not the raw token.

### 2. Restrict HTTP Referer Policy on Reset Pages
Protect sensitive URLs by configuring the `Referrer-Policy` header on all password reset routes:
```http
Referrer-Policy: no-referrer
```
Additionally, avoid loading third-party scripts, analytics tags, or external images on pages where tokens appear in the URL.

### 3. Move Tokens to POST Request Bodies / Fragments
Instead of passing tokens in the query string (`?token=XYZ`), use URL fragments (`#token=XYZ`) or prompt the user to input a code, preventing the token from being sent over the network in the `Referer` header.

### 4. Global Session Revocation on Password Change
Whenever credentials change, increment a `token_version` column in the user database or invalidate all active session keys in Redis, forcing all active devices to re-authenticate.

---

## 📊 CVSS 3.1 Severity Reference

- **Password Reset Poisoning via Host Header (Direct ATO):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H` (Score: **8.8 - High**)
- **Reset Token Leakage via Referer Header to Third Parties:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:N` (Score: **8.1 - High**)
- **Failure to Invalidate Existing Sessions on Password Change:** `CVSS:3.1/AV:N/AC:H/PR:L/UI:N/S:U/C:H/I:L/A:N` (Score: **5.9 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target reset flow verified in scope.
- [ ] Used strictly dedicated test accounts and controlled email inboxes.
- [ ] Tested Host header injection and proxy forwarding headers.
- [ ] Checked for token leakage in `Referer` headers to third-party CDNs.
- [ ] Confirmed single-use and expiry enforcement.
- [ ] Verified that existing sessions are terminated across all devices.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice broken password reset flows, token entropy analysis, and host poisoning:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic) | Practitioner | Tampering with hidden username parameter to reset victim password |
| **PortSwigger Academy** | [Password reset poisoning via Apache / Nginx Host header](https://portswigger.net/web-security/host-header/lab-host-header-basic-password-reset-poisoning) | Practitioner | Poisoning `Host` header to redirect reset link tokens to attacker domain |
| **OWASP** | [crAPI: Password Reset OTP Brute Force](https://github.com/OWASP/crAPI) | Realistic | Resetting other user accounts by brute forcing unthrottled 4-digit OTP |
| **TryHackMe** | [Broken Authentication: Password Reset](https://tryhackme.com/room/brokenauth) | Easy | Referer token leakage, predictable tokens, and session persistence |
