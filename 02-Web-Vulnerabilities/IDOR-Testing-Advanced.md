# 📁 Advanced IDOR & BOLA Testing — Enterprise & API Vectors

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Testing advanced object-level access controls requires strict authorization and rigorous methodology. Test strictly against synthetic resources created under your controlled testing organizations. Never target production accounts or attempt unauthorized multi-tenant cross-tenant pivots.

---

## 📌 Executive Summary

Modern web applications and microservice architectures rarely present simple integer-based IDORs on standard REST routes. Modern access control vulnerabilities manifest as **Broken Object-Level Authorization (BOLA)** across GraphQL schemas, multi-tenant cloud architectures, message queues, export bulk endpoints, and asynchronous worker tasks.

These advanced IDOR patterns occur when authorization controls are applied inconsistently across different protocol layers, API gateways, and microservice RPC calls.

---

## 🧬 Advanced Architectural Attack Vectors

```
                                      +------------------------------------+
                                      |         GraphQL Gateway            |
                                      | (Authorizes root query node ✅)    |
                                      +-----------------+------------------+
                                                        |
                                            Resolves nested edge:
                                            "user -> organization -> members"
                                                        |
                                                        v
                                      +------------------------------------+
                                      |       Downstream Microservice      |
                                      | (Skips authorization on sub-object |
                                      |  yielding Tenant B data! ❌)       |
                                      +------------------------------------+
```

---

## 🔍 Advanced Testing Methodologies

### 1. GraphQL BOLA (Object Traversal & Mutations)
In GraphQL, authorization is frequently checked only on the top-level Query field, leaving child resolvers and mutations unprotected.

#### Nested Node Traversal
```graphql
query GetMyProfile {
  viewer {
    id
    # Viewer is authorized, but child field allows arbitrary ID lookup:
    organization(id: "[CONTROLLED_TENANT_B_ID]") {
      id
      name
      billingEmail
      apiKeys
    }
  }
}
```

#### Mutation Target Swapping
Check mutations where the user inputs an entity ID to update:
```graphql
mutation UpdateTeamSettings {
  updateTeam(input: {
    teamId: "[CONTROLLED_TEAM_B_ID]", # Controlled test object owned by Account B
    role: "Admin",
    notificationEmail: "tester@example.com"
  }) {
    team {
      id
      role
    }
  }
}
```

---

### 2. Multi-Tenant SaaS Isolation Bypasses
In multi-tenant SaaS environments, accounts belong to an `organization_id` or `workspace_id`. Check whether an account in Tenant A can interact with records in Tenant B by manipulating tenant headers or parameters:

#### Tenant ID Parameter Mismatch
Send Account A's valid session token, but swap the workspace identifier:
```http
POST /api/v2/workspaces/billing/cards HTTP/1.1
Host: example.com
Authorization: Bearer [TOKEN_TENANT_A]
X-Workspace-Id: [WORKSPACE_ID_TENANT_B]
Content-Type: application/json

{
  "card_id": "[CARD_ID_TENANT_B]"
}
```

#### Subdomain vs Header Tenant Confusion
If the target uses tenant subdomains (`tenant-a.app.example.com`), send requests to Tenant A's host while requesting Tenant B's objects in the body, or vice-versa.

---

### 3. Bulk & Batch Export Endpoints
Developers often enforce single-record access controls on `/item/{id}`, but overlook authorization when implementing export, batch, or multi-select functionality.

#### Comma-Separated IDs
```http
GET /api/v1/reports/export?ids=[OWN_ID_1],[OWN_ID_2],[TARGET_ID_3] HTTP/1.1
Authorization: Bearer [TOKEN_A]
```
*Test observation:* Does the resulting CSV/PDF contain rows belonging to Account B?

#### JSON Batch Arrays
```json
POST /api/v1/invoices/bulk-download HTTP/1.1
Authorization: Bearer [TOKEN_A]
Content-Type: application/json

{
  "invoice_ids": [
    "[OWN_INVOICE_A1]",
    "[TARGET_INVOICE_B1]"
  ]
}
```

---

### 4. Asynchronous & Background Job IDOR
When an application offloads work to asynchronous message queues (e.g., Celery, RabbitMQ, AWS SQS, Redis workers), authorization checks performed at the API layer are often lost before worker processing.

#### Report Generation Trigger
1. Account A triggers a heavy PDF generation request specifying a target object ID owned by Account B.
2. The API enqueues the job: `enqueue("generate_pdf", {object_id: B_ID, user_id: A_ID})`.
3. The background worker generates the PDF without checking ownership and stores it in a predictable cloud path or emails it to Account A's inbox.

---

### 5. Blind / Write-Only IDOR
Certain actions do not reflect data back into the immediate HTTP response, but alter state or trigger external events:
- **Password reset initiation:** Changing the target user ID in `/api/reset-password` without email verification.
- **Invitations & Collaborator Assignment:** Adding Account A's email to Account B's private project by submitting `{"project_id": "[TARGET_PROJECT_B]"}`.
- **Webhooks & Notification Subscriptions:** Subscribing an attacker-controlled endpoint to receive event webhooks for a victim's organization ID.

---

## 🛠️ Safe Verification & Evidence Collection

When validating advanced BOLA findings:
1. **Never dump foreign records.** Confirm that the first test record triggers unauthorized disclosure, document that single record, and immediately stop.
2. **Redact synthetic test identifiers** in your bug bounty report.
3. **Compare status and response length:**
   ```text
   Test 1 (Own Record ID): 200 OK | Content-Length: 1420 bytes
   Test 2 (Victim Record ID): 200 OK | Content-Length: 1395 bytes -> CONFIRMED BOLA
   Test 3 (Non-existent ID): 404 Not Found | Content-Length: 45 bytes
   ```

---

## 🛡️ Enterprise Remediation Architecture

### 1. Enforce Authorization at the Resolver / Repository Layer
Never rely on API Gateway path filters or GraphQL root query checks alone. Implement field-level authorization inside resolvers:
```javascript
// GraphQL Resolver with Field-Level Authorization Check
const resolvers = {
  Organization: {
    members: async (parent, args, context) => {
      // Ensure the authenticated caller belongs to this organization
      if (context.user.organizationId !== parent.id) {
        throw new ForbiddenError("Not authorized to access members of this organization.");
      }
      return db.getMembers(parent.id);
    }
  }
};
```

### 2. ABAC (Attribute-Based Access Control)
Adopt ABAC or PBAC (Policy-Based Access Control) using engines like **Open Policy Agent (OPA)** or **AWS Verified Permissions** to evaluate:
- Subject (User role, attributes)
- Resource (Owner, classification, tenant)
- Action (Read, Update, Export)
- Environment (Network, IP, time)

---

## ✅ Advanced BOLA Verification Checklist

- [ ] Evaluated all GraphQL queries, mutations, and nested child resolvers.
- [ ] Tested multi-tenant boundary boundaries across subdomains and tenant headers.
- [ ] Checked batch, export, and bulk-processing endpoints with mixed ID lists.
- [ ] Inspected background job and asynchronous report generation triggers.
- [ ] Tested blind write operations (invitations, collaborator additions, webhook setups).
- [ ] Confirmed remediation prevents both horizontal and vertical privilege escalation.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Master advanced authorization bypasses and unpredictable ID techniques:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [User ID controlled by request parameter, with unpredictable user IDs](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-unpredictable-user-ids) | Practitioner | Discovering target GUIDs from public blog profile links |
| **PortSwigger Academy** | [User ID controlled by request parameter with data leakage in redirect](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect) | Practitioner | 302 redirect response body leakage despite client redirect |
| **PortSwigger Academy** | [User ID controlled by request parameter with password disclosure](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure) | Practitioner | Extracting administrator credentials from pre-filled form fields |
| **OWASP** | [crAPI: Mechanic Report Order BOLA](https://github.com/OWASP/crAPI) | Realistic | Bypassing authorization in nested REST & GraphQL endpoints |
| **Hack The Box** | [HTB Machine: Shoppy](https://app.hackthebox.com/machines/Shoppy) | Easy | Bypassing authorization and extracting multi-user orders |
