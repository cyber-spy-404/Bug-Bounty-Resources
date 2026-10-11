# 📁 Cross-Site Request Forgery (CSRF) — Fundamentals & Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Cross-Site Request Forgery exploits an authenticated user's active session. Test strictly on authorized targets. **Never send CSRF exploit proofs-of-concept to real users or staff members.** Always test using two dedicated, controlled test accounts you own, verifying that Account A can trigger state-changing actions on Account B's session.

---

## 📌 Executive Summary

**Cross-Site Request Forgery (CSRF)** is an attack that forces an end user to execute unwanted, unauthorized actions on a web application in which they are currently authenticated. Because web browsers automatically include credentials (such as session cookies, HTTP Basic Auth, and client certificates) with cross-origin requests, a vulnerable web application cannot distinguish between an intentional request initiated by the user and a forged request submitted by a malicious third-party site.

### 🎯 High-Impact Target Actions
CSRF is significant only when it targets **state-changing operations**:
- Modifying email address or phone number (leading directly to Account Takeover).
- Changing account password or security questions.
- Adding an attacker-controlled API key, SSH key, or webhook.
- Transferring funds, placing orders, or modifying shipping addresses.
- Changing privacy settings or granting administrative permissions.

```
+------------------------------------+
| 1. Victim visits Malicious Site    |
|    (https://attacker-site.com)     |
+------------------+-----------------+
                   |
                   | 2. Malicious HTML auto-submits form:
                   |    POST https://bank.example.com/api/transfer
                   |    to="attacker"&amount="500"
                   v
+------------------------------------+
| 3. Victim's Browser                |
|    Automatically attaches victim's |
|    session cookie: "session=XYZ"   |
+------------------+-----------------+
                   |
                   v
+------------------------------------+
| 4. Bank Application Server         |
|    Verifies cookie XYZ (Valid!)    |
|    Executes unauthorized transfer! |
+------------------------------------+
```

---

## 🔍 Core Discovery & Testing Methodology

### Step 1: Identify State-Changing Requests
Filter your proxy HTTP history for `POST`, `PUT`, `DELETE`, or `PATCH` requests that change user state (e.g. updating profile details, changing passwords, modifying permissions).

---

### Step 2: Analyze Current Anti-CSRF Defenses
Inspect the request to see what CSRF protections are in place:
1. **CSRF Token:** Is there an anti-CSRF token in the request body (e.g., `csrf_token`, `_token`, `authenticity_token`) or in an HTTP header (e.g., `X-CSRF-Token`)?
2. **SameSite Cookie Attribute:** Inspect the `Set-Cookie` header in the login response:
   - `SameSite=Strict`: Immune to CSRF in standard navigations.
   - `SameSite=Lax`: Immune to CSRF for cross-origin `POST` requests (with the exception of Chrome's 2-minute Lax window).
   - `SameSite=None`: Vulnerable to cross-site requests if Secure flag is present and tokens are absent.
   - Unspecified: Modern browsers default to `SameSite=Lax`.

---

### Step 3: Test for Common Anti-CSRF Token Flaws

| Test Vector | Testing Technique | Vulnerable Indicator |
| :--- | :--- | :--- |
| **1. Token Omission** | Strip the `csrf_token` parameter completely from the request body. | Server processes request with `200 OK`. |
| **2. Blank Token** | Send the parameter with an empty string: `csrf_token=`. | Server processes request with `200 OK`. |
| **3. Token Length / Arbitrary String** | Replace token with an arbitrary value of the same length or `AAAA...`. | Server validates format but not authenticity. |
| **4. Cross-Account Token Replay** | Generate a valid token with Account A, and submit it in Account B's request. | Server verifies token exists, but does not bind token to user session! |
| **5. Method Swapping (POST -> GET)** | Convert `POST /change-email` into `GET /change-email?email=test@example.com`. | Backend router handles GET using the same handler. |

---

## 🛠️ Safe Educational Proof-of-Concept Template

To demonstrate CSRF safely without attacking third parties, construct an auto-submitting HTML test page in a local test file:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Safe CSRF Demonstration</title>
</head>
<body>
  <h1>CSRF Proof of Concept</h1>
  <p>Verifying lack of anti-CSRF token on email update endpoint...</p>
  
  <form id="csrfForm" action="https://example.com/api/user/email" method="POST">
    <input type="hidden" name="email" value="controlled-test-account@example.com" />
  </form>

  <script>
    // Auto-submits the form when page loads
    document.getElementById('csrfForm').submit();
  </script>
</body>
</html>
```

---

## 💡 High-Value Bug Bounty Tips & Edge Cases

### 1. JSON-to-Form Conversion (Content-Type Spoofing)
If an API accepts JSON (`application/json`), browsers enforce a CORS preflight (`OPTIONS` request) by default. However, test whether the server also parses:
- `application/x-www-form-urlencoded`
- `multipart/form-data`
- `text/plain` (which can be sent cross-origin without triggering preflight!)

### 2. Login CSRF
If the login endpoint lacks CSRF protection, an attacker can log a victim into the attacker's account. When the victim enters sensitive personal data, search queries, or payment cards, it is saved into the attacker's account history.

### 3. Logout CSRF
Logging out a user via CSRF is generally considered low severity (DoS/nuisance), but can be chained with login CSRF to force re-authentication against an attacker-controlled session.

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Synchronizer Token Pattern (Anti-CSRF Tokens)
Generate a cryptographically secure, high-entropy, unpredictable token bound strictly to the user's current session:
```python
# Flask-WTF / Django built-in CSRF protection
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect(app)
```
In templates:
```html
<form method="POST">
  <input type="hidden" name="csrf_token" value="{{ csrf_token() }}"/>
  ...
</form>
```

### 2. Configure `SameSite=Lax` or `SameSite=Strict` on Session Cookies
```http
Set-Cookie: session_id=XYZ12345; Secure; HttpOnly; SameSite=Lax; Path=/
```
- `Strict`: Session cookie is never sent on cross-site requests, providing robust protection.
- `Lax`: Cookie is withheld on cross-site sub-requests (images, scripts, forms), but included on top-level GET navigations.

### 3. Re-Authentication for Sensitive Operations
Require current password confirmation or step-up MFA verification before performing critical account changes (email change, password update, banking transfer).

---

## 📊 CVSS 3.1 Severity Reference

- **CSRF on Email / Password Change (Direct ATO):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H` (Score: **8.8 - High**)
- **CSRF on Financial Transaction / State Modification:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N` (Score: **6.5 - Medium**)
- **CSRF on Low-Impact Settings (Notification Toggle):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:L/A:N` (Score: **4.3 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target endpoint is confirmed in scope and performs a state-changing operation.
- [ ] Confirmed that session cookie does not have `SameSite=Strict`.
- [ ] Tested token omission, blank token, and cross-account token replay.
- [ ] Generated clean, self-contained HTML PoC.
- [ ] Verified that state change actually completed in victim test session.
- [ ] Redacted personal session identifiers from submission report.
