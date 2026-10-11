# 📁 Cookie Security & Session State Testing — Methodology Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Cookie and session state testing evaluates token transmission, persistence, and invalidation. Test strictly on authorized assets. **Never attempt to steal, exfiltrate, or reuse cookies belonging to real users.** Conduct all testing using your own dedicated test sessions.

---

## 📌 Executive Summary

HTTP cookies are the primary mechanism used by web applications to maintain state across stateless HTTP transactions. Because session cookies represent an authenticated user's digital identity, weaknesses in cookie attributes, transmission, or lifecycle management allow attackers to hijack accounts, bypass two-factor authentication, or perform session fixation attacks.

```
+---------------------------------------------------------------------------------+
|                       Cookie Security Architecture                              |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  Security Flags                   Prefix Standards                 Lifecycle State
  - HttpOnly (Blocks XSS theft)    - __Host- (Strict Origin)        - Regeneration on login
  - Secure (HTTPS only)            - __Secure- (HTTPS bound)        - Invalidation on logout
  - SameSite (Lax/Strict)                                           - Expiry enforcement
```

---

## 🔍 Core Cookie Attributes & Security Flags

Every sensitive session cookie returned in the `Set-Cookie` HTTP header must implement the following defensive flags:

| Attribute | Defensive Value | Security Impact if Omitted / Misconfigured |
| :--- | :--- | :--- |
| **`HttpOnly`** | Present | **High:** If omitted, client-side JavaScript can read the cookie via `document.cookie`, allowing any XSS flaw to steal the session. |
| **`Secure`** | Present | **High:** If omitted, the cookie will be transmitted over plaintext HTTP if the user visits an unencrypted link, exposing it to network sniffing. |
| **`SameSite`** | `Lax` or `Strict` | **Medium-High:** If set to `None` without tokens, enables Cross-Site Request Forgery (CSRF). |
| **`Domain`** | Omitted (Host-Only) | If set broadly (e.g. `Domain=.example.com`), any compromised or vulnerable subdomain (`user-blog.example.com`) can read or overwrite the cookie. |
| **`Path`** | `/` or Restricted | Restricts cookie availability to specific URL subpaths. |
| **`Max-Age` / `Expires`**| Explicit timestamp | If omitted, becomes a transient session cookie that may persist if browser state is restored. |

---

## 🛡️ Modern Cookie Prefixes (RFC 6265bis)

Modern browsers support security prefixes that reject cookies that violate strict origin rules:

### 1. `__Host-` Prefix (Highest Protection)
A cookie named with `__Host-` (e.g., `Set-Cookie: __Host-SessionId=XYZ; ...`) must meet all of the following criteria or modern browsers will reject it:
1. It must have the `Secure` flag.
2. It must be sent from a secure HTTPS origin.
3. It must **not** contain a `Domain` attribute (locking it strictly to the current host, preventing subdomain tampering).
4. Its `Path` must be set to `/`.

### 2. `__Secure-` Prefix
A cookie named with `__Secure-` must be sent from an HTTPS origin and contain the `Secure` flag, but can still include a `Domain` attribute.

---

## 💡 High-Value Bug Bounty Attack Vectors

### 1. Session Fixation
Occurs when an application preserves an existing session token across an authentication boundary rather than regenerating a new token upon login:

#### Testing Workflow:
1. Open an incognito browser window. Do not log in.
2. Record the pre-authentication session cookie: `session_id=ANONYMOUS_TOKEN_123`.
3. Submit valid credentials for your controlled test account.
4. Inspect the post-authentication HTTP response.
5. **Vulnerable Indicator:** The server does not issue a new `Set-Cookie` header, and continuing to use `session_id=ANONYMOUS_TOKEN_123` now grants access to the authenticated user account!
6. *Attack Scenario:* An attacker can pre-set a victim's cookie (via CRLF or subdomain tossing) and wait for the victim to log in.

---

### 2. Subdomain Cookie Tossing
If an application defines cookies with a wildcard domain (`Domain=.example.com`), an attacker with access to a secondary or user-controlled subdomain (`evil.example.com`) can "toss" a malicious cookie:
```http
Set-Cookie: session_id=ATTACKER_KNOWN_SESSION; Domain=.example.com; Path=/
```
When the victim visits the main site `https://example.com`, the browser transmits the attacker's tossed cookie, potentially overriding the legitimate session or poisoning CSRF tokens.

---

### 3. Insufficient Session Destruction on Logout
Does clicking "Log Out" truly invalidate the session on the server?
1. Log into your test account and copy the active session cookie value.
2. Click "Log Out" in the application UI.
3. Open a separate terminal or Burp Repeater, and send an authenticated request (e.g. `GET /api/user/profile`) including the copied pre-logout cookie.
4. **Vulnerable Indicator:** The server returns `200 OK` with user data instead of `401 Unauthorized`. The server merely cleared the client-side cookie, leaving the token active in the server session store indefinitely!

---

### 4. Session Persistence Across Password Changes
When a user changes their password or updates security credentials:
1. Active sessions on other browsers or mobile devices should be terminated immediately.
2. Log in on Browser 1 and Browser 2 simultaneously.
3. Change the password on Browser 1.
4. Attempt a protected request on Browser 2 using its existing session token.
5. **Vulnerable Indicator:** Browser 2 remains authenticated and active.

---

## 🛠️ Safe Educational Verification Template

### Verification HTTP Request (Post-Logout Probe)
```http
GET /api/v1/user/settings HTTP/1.1
Host: example.com
Cookie: session_id=[SESSION_TOKEN_RECORDED_PRIOR_TO_LOGOUT]
User-Agent: Mozilla/5.0
```

### Vulnerable Response Body
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "authenticated",
  "email": "controlled-tester@example.com",
  "note": "Session remained active after logout event!"
}
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Enforce Complete Cookie Flags in Production
In framework configuration:
```python
# Django settings.py (SECURE)
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
CSRF_COOKIE_SECURE = True
CSRF_COOKIE_HTTPONLY = True
CSRF_COOKIE_SAMESITE = 'Lax'
SESSION_COOKIE_NAME = '__Host-session' # Use RFC prefix
```

### 2. Regenerate Session Identifiers on Privilege Transition
Always issue a brand-new session ID upon successful login to eliminate session fixation:
```php
// PHP (SECURE)
session_start();
// After validating username and password:
session_regenerate_id(true); // 'true' deletes old session data on server
$_SESSION['user_id'] = $authenticated_user_id;
```

### 3. Server-Side Session Invalidation
On logout, delete the session record directly from the server-side store (Redis, database, memcached):
```javascript
// Express session destruction
app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    res.clearCookie('connect.sid');
    res.json({ message: "Logged out successfully" });
  });
});
```

---

## 📊 CVSS 3.1 Severity Reference

- **Session Fixation leading to Account Hijacking:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:N` (Score: **8.1 - High**)
- **Session Tokens not Invalidated on Logout / Password Reset:** `CVSS:3.1/AV:N/AC:H/PR:L/UI:N/S:U/C:H/I:L/A:N` (Score: **5.9 - Medium**)
- **Missing `HttpOnly` or `Secure` Flag on Session Cookie:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` (Score: **5.3 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Evaluated all flags on sensitive cookies (`HttpOnly`, `Secure`, `SameSite`).
- [ ] Tested session regeneration between pre-login and post-login states.
- [ ] Confirmed whether session tokens remain valid after clicking logout.
- [ ] Tested whether existing sessions survive password change events.
- [ ] Included exact `Set-Cookie` response headers in the disclosure report.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice session cookie manipulation, flag evasion, and SameSite bypasses:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [SameSite Lax bypass via method override](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-lax-bypass-via-method-override) | Practitioner | Bypassing SameSite Lax cookie restrictions via method spoofing |
| **PortSwigger Academy** | [SameSite Strict bypass via client-side redirect](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-client-side-redirect) | Practitioner | Bypassing SameSite Strict via on-site open redirect gadget |
| **TryHackMe** | [Cookie Stealing & Session Hijacking](https://tryhackme.com/room/cookiehijacking) | Easy | Missing `HttpOnly` flag exploitation via XSS payloads |
| **PentesterLab** | [Session Fixation Badge](https://pentesterlab.com/) | Medium | Exploiting persistent session identifiers before and after login |
