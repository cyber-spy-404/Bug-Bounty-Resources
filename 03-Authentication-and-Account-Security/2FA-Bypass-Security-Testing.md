# 📁 Two-Factor Authentication (2FA) Bypasses — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Two-factor authentication (2FA/MFA) protects high-value user accounts. Test strictly against assets covered by explicit authorization. **Never attempt to defeat, bypass, or tamper with active MFA controls on accounts belonging to real people.** Use two dedicated test accounts you control to verify step-up enforcement.

---

## 📌 Executive Summary

**Two-Factor Authentication (2FA)** adds a second layer of defense by requiring users to supply two distinct authentication factors (typically something they know, like a password, and something they possess, like a TOTP authenticator code or SMS OTP) before granting authenticated access.

A **2FA Bypass** occurs when implementation weaknesses in state management, session binding, response handling, or auxiliary workflows allow an attacker with knowledge of only the password (the first factor) to bypass or neutralize the second factor requirement and access the protected account.

```
+---------------------------------------------------------------------------------+
|                         Common 2FA Bypass Vectors                               |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  Forced Browsing / Direct Req.    Response Manipulation            Alternative Flow Bypass
  - Direct GET /dashboard         - Change 401 -> 200 OK           - OAuth / SSO login
  - Direct API requests           - {"success": true}              - Password reset auto-login
```

---

## 🔍 Core Bypass Vectors & Testing Methodology

### 1. Direct Endpoint Access / Forced Browsing
When an application creates the fully authenticated session cookie immediately after the password step, and relies solely on frontend UI routing to display the 2FA prompt:
1. Enter valid username and password on the test account.
2. When presented with the 2FA prompt at `/auth/2fa`, **do not enter a code**.
3. In a separate browser tab, navigate directly to protected endpoints:
   - `GET /dashboard`
   - `GET /account/settings`
   - `GET /api/v1/user/profile`
4. **Vulnerable Indicator:** The server responds with `200 OK` and renders user data. The server trusted the session cookie without checking whether the second factor was completed!

---

### 2. Client-Side Response & Status Code Manipulation
In single-page applications (React, Angular, Vue) where authorization routing is decided on the client:
1. Intercept the 2FA code verification submission in Burp Suite / Caido.
2. Submit an intentionally incorrect OTP code (e.g. `000000`).
3. Intercept the server's response to this request:
   ```http
   HTTP/1.1 401 Unauthorized
   Content-Type: application/json

   {"success": false, "message": "Invalid verification code"}
   ```
4. Tamper with the response in the proxy before it reaches the browser:
   ```http
   HTTP/1.1 200 OK
   Content-Type: application/json

   {"success": true, "message": "Verification successful"}
   ```
5. **Vulnerable Indicator:** If the application transitions to the authenticated dashboard and subsequent API calls succeed, the server relied on the client to enforce 2FA verification!

---

### 3. Alternative Login Channels & Mobile APIs
Web applications frequently enforce 2FA on main desktop web portals, but overlook secondary routes:
- **Mobile Endpoints:** Check mobile-specific APIs (`/api/mobile/login`, `/api/v1/app/authenticate`). Mobile endpoints often skip 2FA or issue permanent bearer tokens with credentials alone.
- **OAuth / Social Logins:** Check if clicking "Sign in with Google" or "Sign in with GitHub" bypasses the 2FA challenge if the email matches.
- **Legacy API Versions:** Test `/api/v1/login` or `/api/v0/authenticate`.

---

### 4. Password Reset Auto-Login Bypass
Test the interplay between password reset and 2FA:
1. Enable 2FA on your test account.
2. Initiate a password reset via email.
3. Complete the password reset by entering a new password.
4. **Vulnerable Indicator:** If the application automatically logs the user in upon completing the password reset without asking for the 2FA code, an attacker who compromises the victim's email can bypass 2FA entirely!

---

### 5. Lack of Code Expiry & Code Reuse
1. Request an OTP code and enter it successfully to complete 2FA.
2. Log out of the account.
3. Log in again with the password, and re-submit the **same OTP code used previously**.
4. **Vulnerable Indicator:** The server accepts the reused code. OTP codes must be strictly single-use and invalidated immediately upon first submission.

---

### 6. Cross-Account OTP Binding Flaw
1. Set up Account A and Account B, both with 2FA enabled.
2. Log in with Account A to trigger a 2FA challenge.
3. Log in with Account B in a separate window to trigger a 2FA challenge.
4. Submit the legitimate OTP generated for Account A into the verification prompt for **Account B**.
5. **Vulnerable Indicator:** The server verifies that the code is valid, but fails to check whether it belongs to Account B's specific user ID!

---

### 7. Client-Controlled Cookie State
Inspect cookies issued during the 2FA challenge:
- Does the server set cookies like `2fa_verified=false`, `mfa_passed=0`, or `account_state=pending`?
- Test changing `2fa_verified=false` to `2fa_verified=true` or `1`.

---

## 🛠️ Safe Educational Verification Template

### Step 1: Password Submission (Step 1 Complete)
```http
POST /api/v1/auth/login HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "email": "controlled-tester@example.com",
  "password": "ValidTestPassword123!"
}
```
*Server Response:*
```http
HTTP/1.1 200 OK
Set-Cookie: session_temp=ABC12345; Path=/; HttpOnly; Secure
Content-Type: application/json

{
  "status": "mfa_required",
  "challenge_id": "CHALLENGE_9921"
}
```

### Step 2: Test Protected API Access directly with `session_temp`
```http
GET /api/v1/user/billing HTTP/1.1
Host: example.com
Cookie: session_temp=ABC12345
```
- **Vulnerable:** Returns `200 OK` with sensitive billing data (2FA not enforced server-side).
- **Secure:** Returns `403 Forbidden` (`{"error": "MFA completion required"}`).

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Implement a Strict Server-Side State Machine
Never issue a fully authenticated session token until both factors are verified. Use distinct session states:
```python
# Django / Flask State Machine Pattern (SECURE)
def handle_password_login(user):
    if user.mfa_enabled:
        # Issue a temporary, restricted token that ONLY allows access to /verify-2fa
        temp_token = create_mfa_pending_token(user.id)
        return {"status": "mfa_required", "token": temp_token}
    else:
        # Full session granted only when MFA is not required
        return {"status": "authenticated", "token": create_full_session_token(user.id)}

def verify_2fa(mfa_token, user_code):
    user_id = validate_mfa_pending_token(mfa_token)
    if not verify_totp(user_id, user_code):
        raise AuthenticationError("Invalid 2FA code")
    
    # Invalidate the temporary token and grant the full session
    invalidate_token(mfa_token)
    return {"status": "authenticated", "token": create_full_session_token(user_id)}
```

### 2. Strictly Enforce Single-Use & Short TTL on OTP Codes
- Expire codes after 3 to 5 minutes maximum.
- Invalidate codes immediately after first submission (regardless of whether subsequent steps succeed or fail).
- Maintain a replay cache of consumed codes in Redis.

### 3. Enforce Rate Limiting & Account Lockout
Implement strict throttling on 2FA endpoints (e.g., maximum 5 attempts per challenge, after which the challenge is invalidated and the user must re-enter their password).

---

## 📊 CVSS 3.1 Severity Reference

- **Complete 2FA Bypass to Full Account Access:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H` (Score: **8.8 - High**)
- **2FA OTP Brute-Force via Missing Rate Limiting:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N` (Score: **8.1 - High**)
- **2FA Bypass on Password Reset Auto-Login:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:N` (Score: **8.1 - High**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Used strictly dedicated test accounts with 2FA enabled.
- [ ] Tested forced browsing to protected endpoints while on the 2FA prompt.
- [ ] Evaluated response status manipulation (401 -> 200).
- [ ] Checked alternative channels (OAuth, mobile APIs, password reset flows).
- [ ] Tested code reuse and cross-account code replay.
- [ ] Included exact request/response flow with redacted test credentials in report.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice Two-Factor Authentication bypasses and multi-factor logic flaws:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [2FA simple bypass](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass) | Apprentice | Directly navigating to `/my-account` without completing 2FA form |
| **PortSwigger Academy** | [2FA broken logic](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic) | Practitioner | Cookie tampering to generate and verify OTP codes for other accounts |
| **PortSwigger Academy** | [2FA bypass using a brute-force attack](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-bypass-using-a-brute-force-attack) | Practitioner | Turbo Intruder multi-threading to brute force 4-digit MFA codes |
| **OWASP** | [crAPI: MFA & OTP Bypass Challenge](https://github.com/OWASP/crAPI) | Realistic | Bypassing OTP verification through lack of rate-limiting |
| **TryHackMe** | [Authentication Bypass: 2FA Room](https://tryhackme.com/room/authenticationbypass) | Easy | Status code tampering and parameter omission in 2FA handlers |
