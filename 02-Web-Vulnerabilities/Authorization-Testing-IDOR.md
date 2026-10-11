# 📁 Authorization Testing — IDOR & Object-Level Access Control

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Test only applications you own or are explicitly authorized to assess. Use separate test accounts and synthetic records. Never use another person’s token, enumerate real users, or access data outside the approved scope.

## 📌 Executive Summary

Authorization vulnerabilities occur when an application authenticates a user but fails to verify whether that user is allowed to access a particular object or perform a specific action. The supplied five-page PDF focuses on changing account identifiers and request-body representations to assess whether object-level access controls are enforced consistently.

## 🔍 Core Testing Methodology

1. **Create controlled test identities**
   - Set up two accounts you control, such as `[Test-Account-A]` and `[Test-Account-B]`.
   - Create harmless test records owned by each account.
   - Use each account’s own authorized session token.

2. **Map object references**
   - Identify endpoints that accept account IDs, user IDs, group names, email addresses, or object identifiers.
   - Record which identity owns each test object and which operations should be allowed.

3. **Check predictable identifiers**
   - With Account A’s token, request only Account A’s test records.
   - Confirm that the server denies access to Account B’s test records.
   - Do not enumerate or access real account identifiers.

4. **Check identifier combinations**
   - Where an endpoint accepts both a group and a user/object identifier, verify that the server checks both values against the authenticated identity.
   - Confirm that changing one field cannot grant access to an object in another group.

5. **Check identifier data types**
   - If the API documents an integer ID, test whether it safely rejects unexpected strings, arrays, objects, or other types.
   - Verify that parsing and validation do not change which object is selected.

6. **Check alternate object references**
   - If the API supports email addresses, group IDs, or nested objects as references, verify that each reference is authorized server-side.
   - Use only addresses and groups belonging to your test accounts.

7. **Check nested and repeated fields**
   - Review how the API handles nested objects, multiple object values, and duplicate keys.
   - Confirm that ambiguous input is rejected or interpreted consistently and cannot select a different owner’s record.

8. **Check token and object binding**
   - Confirm that the authenticated identity is derived from a validated session or token, not from a client-supplied account field alone.
   - Ensure that changing an object identifier does not change the caller’s identity or privileges.

9. **Review encoding and normalization**
   - Test documented encoding and canonicalization boundaries in a lab or approved environment.
   - Verify that the application, proxy, and backend agree on the identifier after decoding and normalization.
   - Avoid trying arbitrary bypass strings against production systems.

10. **Verify impact safely**
    - Use a harmless read-only test record or a reversible, low-risk action.
    - Record whether the server denies the unauthorized operation.
    - Stop immediately if real user data or an unintended object is returned.

11. **Document and retest**
    - Capture the endpoint, caller role, test object ownership, expected result, actual result, and redacted evidence.
    - Retest the fix with both accounts and every relevant endpoint.

## 💡 High-Value Bug Bounty Tips & Edge Cases

The source PDF organizes examples under these headings:

- **Predictable ID:** Check whether account identifiers can be changed while retaining the same authenticated session.
- **ID combination:** Review endpoints that combine a user/account identifier with another path or object identifier.
- **Integer as ID:** Check server-side validation when an identifier is expected to be numeric.
- **Email as user ID:** Verify ownership checks when an email address is used to identify an account.
- **Group ID:** Confirm that a user cannot access another group’s resources merely by changing a group name or ID.
- **Group and user combination:** Verify authorization across both group and user boundaries.
- **Nested object:** Check whether nested identifier structures are validated and authorized consistently.
- **Multiple objects:** Confirm that multiple or duplicate object fields cannot produce ambiguous selection behavior.
- **Predictable token:** The PDF includes a section about token predictability. Treat tokens as secrets and verify their security design; do not guess or brute-force live tokens.
- **Methods 10–20:** The PDF shows a sequence of variations to an account field, including encoded or delimiter-like representations, arrays, and alternate separators. The document does not explain the expected results for these methods. Treat them as prompts to review canonicalization and parser consistency in a controlled lab, not as confirmed bypasses.
- **Source limitation:** The PDF contains request examples but does not document a target implementation, successful exploit results, or detailed remediation. No listed variation should be assumed to work universally.

## 🛠️ Educational Checklists & Safe Examples

### Example: controlled object-access test

Use placeholder endpoints and records that belong to accounts you control.

```http
GET /api/v1/accounts/[TEST-ACCOUNT-A-RECORD] HTTP/1.1
Host: example.com
Authorization: Bearer [ACCOUNT-A-TEST-TOKEN]
```

Then, as a separate test, request a harmless record owned by Account B while authenticated as Account A:

```http
GET /api/v1/accounts/[TEST-ACCOUNT-B-RECORD] HTTP/1.1
Host: example.com
Authorization: Bearer [ACCOUNT-A-TEST-TOKEN]
```

**Expected secure result:** The second request is denied or returns only data that Account A is explicitly authorized to see. Do not substitute real users’ identifiers or tokens.

### Example: validate identifier types

```json
{
  "account_id": "controlled-test-record"
}
```

```json
{
  "account_id": 12345
}
```

The expected type depends on the API contract. A type mismatch must not cause the server to select an unintended object or bypass authorization.

### Finding template

```text
Target: [Your-Domain]
Endpoint: [In-Scope-Endpoint]
Caller: [Test-Account-A]
Target object owner: [Test-Account-B or Test-Group-B]
Object reference type: [Integer / UUID / Email / Group / Nested Object]
Expected result: [Denied]
Observed result: [Redacted outcome]
Impact: [Demonstrated cross-account access, if any]
Evidence: [Redacted request/response]
```

### Authorization checklist

- [ ] Every object access checks the authenticated caller’s permissions.
- [ ] Account IDs supplied by the client are not treated as proof of identity.
- [ ] Group and user permissions are both enforced where applicable.
- [ ] Email-based lookups verify ownership and authorization.
- [ ] Numeric, string, array, and object inputs are validated against the API schema.
- [ ] Nested objects and duplicate keys are handled consistently.
- [ ] Tokens are unpredictable, securely stored, validated, and not exposed.
- [ ] Identifier decoding and normalization are consistent across application layers.
- [ ] Unauthorized requests do not reveal object data through errors or metadata.
- [ ] Tests use only controlled accounts and synthetic records.

## 🛡️ Mitigation & Remediation

- **Enforce object-level authorization:** On every request, derive the caller from a validated session or token and verify that the caller has permission to access the requested object.
- **Scope database queries to the caller:** Where appropriate, query objects through the authenticated user’s authorized dataset rather than fetching arbitrary objects solely by client-provided ID.
- **Validate identifiers strictly:** Enforce documented types, formats, lengths, and allowed values. Reject malformed or ambiguous references.
- **Authorize every relationship:** For resources tied to users, organizations, or groups, verify all relevant ownership and membership relationships server-side.
- **Do not trust client-supplied identity:** Fields such as `account_id`, email, group name, or user ID are selectors, not evidence of authorization.
- **Use secure tokens:** Generate session and access tokens with suitable cryptographic randomness, validate them correctly, and protect them as secrets.
- **Canonicalize consistently:** Decode and normalize identifiers in a single, well-defined manner before validation and authorization. Avoid different interpretations between proxies and application components.
- **Reject ambiguous structures:** Define how duplicate keys and repeated identifiers are handled; preferably reject ambiguous request bodies.
- **Minimize error disclosures:** Return safe errors for unauthorized access without exposing whether sensitive objects exist, unless the API contract explicitly requires that information.
- **Add regression tests:** Test read, update, delete, and action endpoints using multiple roles and objects owned by different test accounts.

## 📚 Source Notes

This guide is based on the supplied five-page *authorization.pdf*. Page 1 covers predictable IDs, combined identifiers, integer IDs, and email-based user IDs. Page 2 covers group IDs, group/user combinations, nested objects, and multiple objects. Pages 3–5 show additional numbered variations to account-field values and representations.

The source does not explain the expected result of every variation or demonstrate that any specific application is vulnerable. This rewrite preserves the source’s categories, groups the later variations into broader validation concerns, and adds general defensive guidance without presenting them as confirmed exploit techniques.

*Educational use only. Test ethically, stay within scope, and disclose findings through the program’s approved process.*
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice vertical and horizontal role-based privilege escalation:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Unprotected admin functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality) | Apprentice | Accessing privileged administrator endpoints via path discovery |
| **PortSwigger Academy** | [User role controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter) | Practitioner | Elevating privileges by tampering with hidden role parameters |
| **PortSwigger Academy** | [User role can be modified in user profile](https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile) | Practitioner | JSON role elevation (`roleid: 2`) during profile update |
| **TryHackMe** | [Authorization Flaws Room](https://tryhackme.com/room/brokenauth) | Medium | Vertical escalation from regular user to super-administrator |
| **Hacker101 CTF** | [Micro-CMS v2](https://ctf.hacker101.com/) | Medium | Bypassing role-based authentication to edit pages |
