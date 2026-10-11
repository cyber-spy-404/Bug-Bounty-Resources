# 📁 Application-Level Denial of Service (DoS) — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Critical ethics and safety rule:** Network-level volumetric Denial-of-Service (DDoS), SYN flooding, and brute-force traffic flooding are **strictly prohibited** in bug bounty programs and ethical security research. Testing must focus solely on **application-level algorithmic and logical flaws** that cause disproportionate backend resource exhaustion using **a single or minimal number of controlled requests**. Never degrade service availability for production users.

---

## 📌 Executive Summary

**Application-Level Denial of Service (Layer 7 DoS)** occurs when an attacker exploits algorithmic inefficiencies, unbounded resource consumption, or parsing vulnerabilities in backend code to exhaust CPU, memory, database connections, or disk space using a tiny fraction of network bandwidth.

Unlike network-layer volumetric floods, application DoS flaws are architectural software vulnerabilities that can often be demonstrated with a single request.

```
Attacker sends: 1 single HTTP request (500 bytes)
                |
                v
        +----------------------------+
        | Backend Parser / Algorithm |
        | Catastrophic Backtracking  |
        | Or Billion Laughs Expansion|
        +-------------+--------------+
                      |
                      v
        CPU Spikes to 100% for 60 seconds!
        Worker process thread locks! ❌
```

---

## 🔍 Core Application DoS Vulnerability Classes

### 1. Regular Expression Denial of Service (ReDoS)
Occurs when an application evaluates user-supplied input against a poorly crafted regular expression containing nested quantifiers or overlapping alternation, leading to **catastrophic backtracking**.

#### Dangerous Regex Patterns:
- `(a+)+$`
- `([a-zA-Z]+)*$`
- `(a|aa)+$`
- Email/URL regexes with ambiguous grouping: `^([a-zA-Z0-9_\-\.]+)@((\[[0-9]{1,3}\...`

#### Safe Testing Workflow:
1. Identify input fields evaluated by custom regexes (usernames, search filters, URL validators).
2. Test input with increasing lengths of the repeating character without triggering timeout:
   - Length 10: `aaaaaaaaaa!` (Measure response time: 2ms)
   - Length 15: `aaaaaaaaaaaaaaa!` (Measure response time: 20ms)
   - Length 20: `aaaaaaaaaaaaaaaaaaaa!` (Measure response time: 200ms)
3. If response time grows **exponentially** ($O(2^n)$) rather than linearly ($O(n)$), the ReDoS is confirmed. **Stop immediately**—do not send 50+ characters which would lock the server thread.

---

### 2. XML Entity Expansion (Billion Laughs / XML Bomb)
When an application parses XML documents with external entity resolution or recursive entity expansion enabled:
```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
 <!ENTITY lol "lol">
 <!ELEMENT lolz (#PCDATA)>
 <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
 <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
 <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
]>
<lolz>&lol3;</lolz>
```
*Safe Testing:* Keep entity levels minimal (`lol2` or `lol3` max). If the server takes noticeable time to parse a 1KB XML file, entity expansion is unconstrained.

---

### 3. GraphQL Query Depth & Batching Abuse
In GraphQL APIs without query complexity or depth limits:
- **Circular Relation Loops:**
  ```graphql
  query {
    author {
      books {
        author {
          books {
            author {
              books { id }
            }
          }
        }
      }
    }
  }
  ```
- **Query Batching:** Sending an array of 500 heavy queries inside a single HTTP request:
  ```json
  [
    {"query": "{ heavyAnalyticsReport { id } }"},
    {"query": "{ heavyAnalyticsReport { id } }"},
    ...
  ]
  ```

---

### 4. Unbounded Pagination & Database Query Exhaustion
When an application accepts user-controlled pagination parameters without maximum thresholds:
```http
GET /api/v1/users?limit=10000000&offset=0 HTTP/1.1
GET /api/v1/transactions?sort_by=unindexed_column&export=true HTTP/1.1
```
*Test observation:* Does the server attempt to load millions of records into memory at once, resulting in database lockouts or HTTP 504 Gateway Timeouts?

---

### 5. Image & Compression Bombs (Decompression Bombs)
- **Pixel Flooding:** Submitting an image file with tiny byte size (e.g., 5KB PNG) whose header declares extreme dimensions (`100,000 x 100,000` pixels). When parsed by libraries like GD or ImageMagick, the server allocates gigabytes of RAM attempting to render the bitmap canvas.
- **Zip Decompression Bombs:** Archive files that expand from a few kilobytes into hundreds of gigabytes when extracted by backend file ingestion systems.

---

## 🛠️ Safe Educational Verification Template

### Safe ReDoS Exponential Growth Log
```text
Payload A: "a"*15 + "!" -> Duration: 15 ms
Payload B: "a"*18 + "!" -> Duration: 120 ms
Payload C: "a"*21 + "!" -> Duration: 980 ms
Payload D: "a"*24 + "!" -> Duration: 7,840 ms
Conclusion: Exponential growth verified. Testing halted safely to avoid service disruption.
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Adopt Linear-Time Regex Engines (e.g., RE2, Rust regex)
Replace backtracking regex engines with linear-time automata engines that guarantee $O(n)$ matching regardless of input:
```python
# Use Google's re2 library instead of Python's standard 're'
import re2

pattern = re2.compile(r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$")
```

### 2. Disable DTDs and External Entities in XML Parsers
```python
# DefusedXML in Python (Prevents XML Bombs and XXE)
import defusedxml.ElementTree as ET

tree = ET.fromstring(xml_data)
```

### 3. Enforce GraphQL Query Cost & Depth Analysis
```javascript
// GraphQL Depth & Complexity Limiting
import depthLimit from 'graphql-depth-limit';
import { createComplexityLimitRule } from 'graphql-validation-complexity';

const server = new ApolloServer({
  schema,
  validationRules: [
    depthLimit(5), // Reject queries nested more than 5 levels deep
    createComplexityLimitRule(1000) // Assign cost points per query
  ]
});
```

### 4. Strict Resource Allocation Caps
- Enforce strict upper bounds on pagination: `limit = Math.min(requested_limit, 100)`.
- Set maximum image canvas limits before decompression.
- Implement aggressive timeouts on database queries (e.g., `statement_timeout = 3000ms`).

---

## 📊 CVSS 3.1 Severity Reference

- **Single-Request Unauthenticated DoS (Server Crash / Lockout):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H` (Score: **7.5 - High**)
- **Authenticated Algorithmic DoS (High CPU):** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:N/A:H` (Score: **6.5 - Medium**)
- **Low-Impact ReDoS on Minor Input Field:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:L` (Score: **5.3 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Evaluated solely application-level / algorithmic logic flaws.
- [ ] Sent no flood traffic or volumetric requests.
- [ ] Confirmed exponential time increase with small sample iterations.
- [ ] Halted testing immediately before causing real denial of service.
- [ ] Provided exact regex, schema query, or calculation in the report.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice identifying application-level Denial of Service without disrupting production:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Server-side prototype pollution leading to DoS](https://portswigger.net/web-security/prototype-pollution/server-side) | Practitioner | Polluting Object.prototype in Node.js causing server crash |
| **PortSwigger Academy** | [Exploiting XXE to perform SSRF & DoS](https://portswigger.net/web-security/xxe) | Practitioner | Billion laughs XML entity expansion vulnerability |
| **TryHackMe** | [ReDoS Room](https://tryhackme.com/room/redos) | Medium | Catastrophic regex backtracking on string input |
| **OWASP** | [Juice Shop: Zero-Star Feedback ReDoS Challenge](https://owasp.org/www-project-juice-shop/) | Medium | Triggering CPU resource exhaustion via regex payload |
