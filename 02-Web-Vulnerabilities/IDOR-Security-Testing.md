# 📁 Insecure Direct Object References (IDOR) — Comprehensive Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** IDOR vulnerabilities frequently involve customer data and administrative controls. Conduct testing strictly within authorized program scopes. Never access, modify, or delete data belonging to other real users. Always use two dedicated, controlled test accounts (`[Test-Account-A]` and `[Test-Account-B]`) with non-sensitive synthetic test data.

---

## 📌 Executive Summary

**Insecure Direct Object Reference (IDOR)** occurs when an application exposes a reference to an internal object (such as a database key, user ID, invoice number, or filename) in a request parameter, and relies on that parameter to retrieve or alter the resource without validating whether the authenticated user has legitimate authorization to access it.

### 🏗️ Why IDOR Happens
In modern architectures, developers frequently decouple authentication from authorization:
1. The user logs in and receives a valid session token (Authentication ✅).
2. The frontend requests `/api/documents/1042`.
3. The backend code runs a direct query: `SELECT * FROM documents WHERE id = 1042;` without checking `AND tenant_id = current_user.tenant_id` (Authorization ❌).

```
+------------------+         GET /api/invoices/9050         +--------------------+
|  User A Session  | -------------------------------------> |  API Gateway / App |
| (Owns Inv #9048) |                                        | (Verifies Token A) |
+------------------+                                        +---------+----------+
                                                                      |
                                           Direct lookup without owner check:
                                           "SELECT * FROM invoices WHERE id = 9050"
                                                                      |
                                                                      v
                                                            +--------------------+
                                                            | Database / Storage |
                                                            | (Returns Inv #9050 |
                                                            |  belonging to B!)  |
                                                            +--------------------+
```

---

## 🔍 Core Testing Methodology (The Dual-Account Matrix)

Testing for IDOR requires at least two distinct user accounts with the same privilege tier (Horizontal IDOR) or different privilege tiers (Vertical IDOR).

### Step 1: Establish Controlled Test Identities
- **Account A (Attacker Persona):** Session Token `Token_A`, Owns Record `ID_A`.
- **Account B (Victim Persona):** Session Token `Token_B`, Owns Record `ID_B`.

### Step 2: Map Direct Object References
Identify every place where an identifier is supplied by the client:
- **URL Path parameters:** `/api/v2/users/5821/profile`
- **Query strings:** `/download?file_id=8831&account=A9`
- **Request bodies (JSON / Form):** `{"order_id": 4012, "action": "cancel"}`
- **HTTP Headers:** `X-User-Id: 5821`

### Step 3: Execute the Cross-Account Swap Matrix
For every identified endpoint, run the following tests:

| Test Case | Session Token Used | Target Object ID | Expected Secure Response | Vulnerable Indicator |
| :--- | :--- | :--- | :--- | :--- |
| **Baseline Own Access** | `Token_A` | `ID_A` | `200 OK` with Account A data | Normal functionality |
| **Horizontal Read** | `Token_A` | `ID_B` | `403 Forbidden` / `404 Not Found` | `200 OK` returning Account B data |
| **Horizontal Write/Update**| `Token_A` | `ID_B` (PUT/POST/PATCH) | `403 Forbidden` | `200 OK` and Account B record modified |
| **Horizontal Delete** | `Token_A` | `ID_B` (DELETE) | `403 Forbidden` | `200 OK` / `204 No Content` and record purged |
| **Unauthenticated Probe** | None | `ID_A` | `401 Unauthorized` | `200 OK` (Bypasses auth entirely) |

---

## 💡 High-Value Bug Bounty Tips & Subtle Bypasses

When straightforward ID swapping returns `403 Forbidden`, test these field-proven bypass vectors:

### 1. HTTP Verb & Method Tampering
Applications often enforce authorization middleware on `GET` and `POST`, but neglect alternate methods handled by the backend:
- Try changing `GET` to `POST`, `PUT`, `PATCH`, `OPTIONS`, or `HEAD`.
- Test method override headers:
  ```http
  POST /api/invoices/9050 HTTP/1.1
  X-HTTP-Method-Override: GET
  ```

### 2. Parameter Pollution (HPP)
Web servers and application frameworks parse duplicate parameters differently (taking the first, last, or combining into an array):
```http
GET /api/documents?doc_id=OWN_ID&doc_id=TARGET_ID HTTP/1.1
GET /api/documents?doc_id=TARGET_ID&doc_id=OWN_ID HTTP/1.1
```

### 3. Array & JSON Encapsulation
If the server validates single integer parameters but fails on lists, wrap the target ID in an array or object:
```json
// Original request
{"user_id": 1052}

// Bypasses to test
{"user_id": [1052]}
{"user_id": {"id": 1052}}
{"user_id": "1052"}
```

### 4. Legacy API Version Downgrades
Organizations often patch authorization on `/api/v3/` or `/api/v2/`, while leaving older endpoints unpatched:
- Replace `/api/v3/orders/9050` with `/api/v1/orders/9050` or `/api/v0/orders/9050`.
- Look for mobile-specific routes: `/mobile/api/orders/9050` or `/internal/orders/9050`.

### 5. Extension Suffix Bypasses
Certain web application frameworks interpret file extensions by routing requests to different serialization handlers:
```http
GET /api/users/9050.json HTTP/1.1
GET /api/users/9050.xml HTTP/1.1
GET /api/users/9050.html HTTP/1.1
```

### 6. Numeric vs Hashed vs UUID IDs
- **UUIDs/GUIDs:** Even if IDs are unpredictable UUIDs (`550e8400-e29b-41d4-a716-446655440000`), it is still an IDOR if knowing the UUID allows access without permission. Look for leaked UUIDs in public APIs, autocomplete endpoints, or chat rooms.
- **Base64 / Hex Encoded IDs:** Always check if a seemingly complex string is simply an encoded integer: `ODgyMQ==` -> base64 decode -> `8821`.

---

## 🛠️ Safe Educational Demonstration Template

### Safe Testing HTTP Request
```http
POST /api/v2/workspaces/billing HTTP/1.1
Host: example.com
Authorization: Bearer [TOKEN_CONTROLLED_ACCOUNT_A]
Content-Type: application/json

{
  "account_id": "[CONTROLLED_ACCOUNT_B_ID]",
  "action": "view_invoice"
}
```

### Vulnerable Response
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "success",
  "invoice_id": "INV-2026-9042",
  "owner": "[CONTROLLED_ACCOUNT_B_EMAIL]",
  "balance": 1500.00
}
```

### Secure Response (Expected)
```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "error": "access_denied",
  "message": "User does not have permission to view this resource."
}
```

---

## 🛡️ Mitigation & Remediation

Developers should never rely on client-supplied identifiers alone. Enforce object-level access controls at the data layer:

### 1. Context-Bound Database Queries
Always scope queries directly to the authenticated session's user ID or organization ID:
```python
# ❌ VULNERABLE: Direct lookup by client-provided ID
def get_invoice(invoice_id):
    return Invoice.query.filter_by(id=invoice_id).first()

# ✅ SECURE: Query scoped strictly to the authenticated user's organization
def get_invoice(invoice_id, current_user):
    return Invoice.query.filter_by(
        id=invoice_id, 
        organization_id=current_user.organization_id
    ).first_or_404()
```

### 2. Centralized Authorization Policies
Use framework-level policy middleware (e.g., Rails Pundit, Laravel Policies, Spring Security `@PreAuthorize`):
```java
// Spring Security Policy Example
@PreAuthorize("@securityService.canAccessDocument(authentication, #documentId)")
@GetMapping("/documents/{documentId}")
public Document getDocument(@PathVariable Long documentId) {
    return documentService.findById(documentId);
}
```

### 3. Indirect / Session-Scoped Reference Maps
Use per-session random reference mappings (e.g., displaying records as `1, 2, 3` in the UI mapped server-side to internal IDs) or cryptographically signed tokens where client-side references cannot be avoided.

---

## 📊 CVSS 3.1 Severity Scoring Guide

- **Read-Only Sensitive Data (PII, Financial):** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N` (Score: **6.5 - Medium**)
- **State-Changing / Write / Account Modification:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N` (Score: **8.1 - High**)
- **Unauthenticated IDOR with Mass Exfiltration / Full DB Access:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` (Score: **9.1 - Critical**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Used strictly two controlled, approved test accounts.
- [ ] No personal information or third-party user data was accessed or exfiltrated.
- [ ] Confirmed that `Token A` cannot view, edit, or delete `Object B`.
- [ ] Tested all HTTP verbs (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`).
- [ ] Tested parameter pollution and alternate encoding formats.
- [ ] Included exact request/response evidence with redacted test tokens in the report.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice Insecure Direct Object References in authentic application workflows:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Insecure direct object references](https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references) | Apprentice | Download private chat transcripts via static file parameter |
| **PortSwigger Academy** | [User ID controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter) | Apprentice | Horizontal IDOR accessing another customer's account API key |
| **TryHackMe** | [IDOR Room](https://tryhackme.com/room/idor) | Easy | Step-by-step parameter tampering on invoices and user IDs |
| **OWASP** | [crAPI: Vehicle Location BOLA Challenge](https://github.com/OWASP/crAPI) | Realistic | Broken Object Level Authorization accessing other users' cars |
| **Hack The Box** | [HTB Academy: Web Attacks -> IDOR](https://academy.hackthebox.com/) | Medium | Mass parameter tampering, UUID prediction, and chained IDOR |
