# 📁 Authentication Endpoint Testing — Input Validation & Robustness

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Use these test cases only on systems you own or are explicitly authorized to assess. The source PDF contains a large catalogue of login-request variations. This rewrite groups those cases by validation category and uses benign placeholders rather than reproducing authentication-bypass or injection payloads.

## 📌 Executive Summary

Authentication endpoints must handle malformed, unexpected, and unusual input consistently without granting unauthorized access or exposing sensitive details. The supplied 17-page PDF presents 97+ JSON and credential variations; this guide organizes the visible cases into a practical, non-destructive validation workflow instead of treating any single input variation as a universal bypass.

## 🔍 Core Testing Methodology

1. **Establish a baseline**
   - Use a dedicated test account and the documented request format.
   - Record the expected success and failure behavior, status code, response structure, and session state.

2. **Test ordinary and empty values**
   - Check missing, empty, and `null` fields.
   - Check whitespace-only values and leading/trailing whitespace.
   - Confirm that required fields are validated consistently.

3. **Test JSON data types**
   - Where the API expects strings, check how it handles numbers, booleans, arrays, objects, and nested objects.
   - Verify that the server rejects incompatible types rather than coercing them unpredictably.

4. **Test JSON syntax and structure**
   - In a test environment, check malformed JSON, missing separators, trailing commas, extra keys, missing keys, repeated keys, and unexpected nesting.
   - Confirm that invalid bodies fail safely and do not reach authentication logic in an ambiguous state.

5. **Test key names and field mapping**
   - Check expected field names, case sensitivity, swapped fields, missing username/password fields, and unexpected keys.
   - Ensure that the server uses a strict, documented schema and does not silently interpret a different field as an identity or secret.

6. **Test string normalization**
   - Check special characters, quotes, backslashes, escape sequences, newlines, tabs, control characters, Unicode, long strings, and multilingual text.
   - Verify that normalization is consistent and that different representations do not cause identity confusion.

7. **Test numeric and format edge cases**
   - Check numeric strings, leading zeros, negative values, exponent notation, hexadecimal-looking strings, URL-like values, email-like values, IP-like values, and date-like values where relevant.
   - Confirm that the API applies the expected type and format validation instead of relying on implicit conversion.

8. **Test encoding boundaries**
   - In a controlled environment, check encoded values and invalid encoding sequences.
   - Verify that decoding occurs once at the correct layer and that validation is applied to the canonical value.

9. **Test duplicate and unusual keys**
   - Check repeated JSON keys and unusual key names only in a lab or authorized environment.
   - Confirm that the parser and downstream components agree on how duplicate or invalid keys are handled.

10. **Check authentication and session outcomes**
    - For every invalid test case, verify that no authenticated session is created and no protected resource becomes accessible.
    - A `200` response or generic message is not proof of successful or failed authentication by itself.

11. **Record and retest**
    - Group results by behavior instead of listing every payload independently.
    - Save redacted request/response evidence and retest the fix with a regression suite.

## 💡 High-Value Bug Bounty Tips & Edge Cases

The source PDF's examples span the following categories. These are validation classes to investigate, not guaranteed bypasses:

- **Basic and empty credentials:** ordinary values, empty strings, and missing values.
- **Null values:** explicit JSON `null` for fields expected to be strings.
- **Type confusion:** numeric, boolean, array, object, and nested-object values in credential fields.
- **Special characters:** punctuation, quotes, slashes, brackets, braces, parentheses, and other symbols.
- **Injection-shaped input:** the source includes SQL-, HTML-, and JSON-injection-themed examples. Use harmless marker strings in authorized tests and verify that input is treated as data, not executable syntax.
- **Unicode and escapes:** Unicode escapes, multilingual strings, whitespace variants, zero-width characters, control characters, and escape sequences.
- **Length boundaries:** long strings, unusually long keys, overlong values, and very large numbers.
- **Malformed JSON:** missing braces, extra commas, unsupported comments, trailing characters, and invalid separators.
- **Field mapping:** missing username or password keys, swapped keys, unexpected keys, and case-variation tests.
- **Duplicate keys:** repeated field names may be interpreted differently by parsers. Reject duplicates or define one unambiguous behavior.
- **Encoding and representation:** hexadecimal-looking strings, Base64-looking values, scientific notation, leading zeros, and octal-looking strings.
- **Nested data:** arrays or objects nested inside credential fields and additional unexpected objects.
- **URL/email/IP/date formats:** confirm that format-looking strings are validated according to the field's actual contract.
- **Writeup examples:** the last page includes several alternate username representations. Treat these as a prompt to test canonicalization and account-name consistency—not as a list of commands to run against real accounts.
- **Source limitations:** the PDF is largely a payload catalogue and does not document expected results, the target implementation, or evidence that any case bypasses authentication.

## 🛠️ Educational Checklists & Safe Examples

### Benign JSON type-validation examples

Use synthetic values with a test endpoint. These examples are designed to check schema enforcement, not to bypass login.

```json
{
  "username": "controlled-test-user",
  "password": "invalid-test-value"
}
```

```json
{
  "username": 12345,
  "password": false
}
```

```json
{
  "username": null,
  "password": []
}
```

```json
{
  "username": {
    "value": "controlled-test-user"
  },
  "password": "invalid-test-value",
  "unexpected_field": "test-marker"
}
```

### Malformed-body examples

These are intentionally invalid JSON. Use them only to verify safe parser errors in a lab:

```text
{"username": "test-user", "password": "test-value"
```

```text
{"username": "test-user",, "password": "test-value"}
```

### Test record template

```text
Target: [Your-Domain]
Endpoint: [In-Scope-Authentication-Endpoint]
Expected content type: [Expected-Content-Type]
Test category: [Type / Encoding / Syntax / Field Mapping / Length]
Input class: [Synthetic Test Case]
Expected result: [Safe Validation Error / Authentication Failure]
Observed result: [Redacted Response Summary]
Session created: [Yes/No]
Protected access granted: [Yes/No]
Evidence: [Redacted Request/Response]
```

### Validation checklist

- [ ] Required fields are present and have the documented types.
- [ ] Empty, missing, and null values are handled safely.
- [ ] Arrays and objects are rejected when scalar strings are required.
- [ ] Malformed JSON receives a safe client error.
- [ ] Unexpected and duplicate keys are rejected or handled unambiguously.
- [ ] Case sensitivity and field mapping are documented and consistent.
- [ ] Unicode, escape sequences, and control characters do not cause identity confusion.
- [ ] Length limits are enforced before expensive processing.
- [ ] Inputs are treated as data, not executable SQL, HTML, or query syntax.
- [ ] Failed authentication never creates an authenticated session.
- [ ] Error responses do not expose secrets, stack traces, or internal parser details.
- [ ] Tests use only controlled accounts and synthetic data.

## 🛡️ Mitigation & Remediation

- **Enforce a strict schema:** Require the documented fields and types. Reject unexpected types, nested structures, unknown keys, and malformed request bodies where appropriate.
- **Use a reliable JSON parser:** Reject malformed JSON and define a consistent policy for duplicate keys. Ensure upstream proxies, frameworks, and application code interpret the same request consistently.
- **Avoid implicit coercion:** Do not silently convert booleans, arrays, objects, or arbitrary numeric forms into usernames or passwords.
- **Normalize carefully:** Define a documented canonicalization policy for identifiers. Apply it consistently during registration, login, recovery, and account lookup; avoid transformations that merge distinct accounts.
- **Parameterize database operations:** Use prepared statements or safe query APIs. Do not concatenate credential input into SQL or other query languages.
- **Encode output by context:** If input is ever displayed, use context-appropriate output encoding. Input validation alone does not prevent XSS.
- **Set input size limits:** Limit body size, field length, nesting depth, and processing time to reduce parser abuse and denial-of-service risk.
- **Fail closed:** Invalid or ambiguous input must never result in an authenticated session or privileged action.
- **Use generic external errors:** Return safe validation or authentication errors while keeping detailed diagnostics in access-controlled logs.
- **Test authentication state explicitly:** Regression tests should verify session creation and protected-resource access, not just response codes or messages.
- **Automate regression coverage:** Maintain a small, representative suite of type, syntax, encoding, length, and field-mapping cases for every authentication endpoint.

## 📚 Source Notes

This guide is based on the supplied 17-page *authentication.pdf*. The document is titled around JSON tests for authentication endpoints and presents a numbered catalogue of credential variations across pages 1–17. Visible categories include basic and empty credentials, nulls, numbers, booleans, arrays, objects, special characters, SQL/HTML/JSON-themed strings, Unicode and escape sequences, malformed JSON, missing or repeated keys, numeric formats, long values, multilingual input, and alternate username representations.

The source does not provide a target application's implementation, expected outcomes, or evidence that the listed cases work as bypasses. This rewrite groups the cases into validation categories and adds general defensive guidance; it does not claim that any specific target is vulnerable.

*Educational use only. Test ethically, stay within scope, and disclose findings through the program’s approved process.*
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice brute-force defense evaluation, credential stuffing, and response timing:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Username enumeration via different responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses) | Apprentice | Differentiating valid accounts via error message differences |
| **PortSwigger Academy** | [Username enumeration via response timing](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing) | Practitioner | Identifying valid usernames based on server cryptographic delay |
| **PortSwigger Academy** | [Broken brute-force protection, IP block](https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block) | Practitioner | Interleaving legitimate logins to reset IP failed-attempt counters |
| **TryHackMe** | [Hydra Room](https://tryhackme.com/room/hydra) | Easy | Automating web login brute forcing and evaluating defense thresholds |
| **OWASP** | [Juice Shop: Login Bypasses](https://owasp.org/www-project-juice-shop/) | Easy | Testing authentication endpoints for SQLi and logic loopholes |
