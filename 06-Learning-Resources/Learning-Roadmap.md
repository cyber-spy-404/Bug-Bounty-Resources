# 📁 Bug Bounty Hunter Learning Roadmap

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Ethics and Scope:** This roadmap is designed for defensive security education, ethical security research, and authorized bug bounty participation. All techniques must be practiced exclusively in controlled laboratory environments or against bug bounty targets with explicit written authorization.

---

## 📌 Executive Summary

Becoming an effective bug bounty hunter requires a structured, multi-disciplinary progression. Rather than jumping directly into high-complexity exploitation, successful security researchers build strong mental models around how web applications, HTTP protocols, authentication mechanisms, and frameworks are constructed, and systematically identify where developers make implicit assumptions.

This learning roadmap provides a structured, 6-phase journey from web fundamentals to advanced vulnerability research, vulnerability chaining, and professional responsible disclosure.

---

## 🗺️ The 6-Phase Learning Roadmap

```
+-------------------------------------------------------------------+
| Phase 1: Web & Networking Foundations                             |
| (HTTP/S, Browsers, SOP/CORS, Request/Response, Proxies)           |
+---------------------------------+---------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| Phase 2: Reconnaissance & Attack Surface Mapping                  |
| (Target Scopes, Subdomains, ASN/IP, Dorking, Passive Recon)       |
+---------------------------------+---------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| Phase 3: Authentication & Account Security Deep Dive               |
| (Registration, Login, Password Reset, MFA/2FA, ATO, Session Mgmt) |
+---------------------------------+---------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| Phase 4: Core Web & API Vulnerabilities                           |
| (IDOR, XSS, CSRF, File Upload, Host Header, Mass Assignment)      |
+---------------------------------+---------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| Phase 5: Business Logic, Access Controls & Technology Reviews     |
| (Logic Flaws, 403 Bypasses, Rate Limiting, Django/Symfony/Jira)   |
+---------------------------------+---------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| Phase 6: Advanced Research, Chaining & Professional Reporting     |
| (RCE Classes, Vulnerability Chaining, CVSS, Triage Communication) |
+-------------------------------------------------------------------+
```

---

## 🧭 Detailed Roadmap Phases

### Phase 1: Foundations & Tooling
*Objective: Understand how the modern web works under the hood before looking for flaws.*

- **Protocols & Networking:**
  - HTTP/1.1 vs HTTP/2 vs HTTP/3 request/response structures.
  - Headers, cookies, caching directives, and status codes (2xx, 3xx, 4xx, 5xx).
  - TCP/IP basics, DNS resolution, and TLS handshakes.
- **Browser Security Architecture:**
  - Same-Origin Policy (SOP) and Cross-Origin Resource Sharing (CORS).
  - DOM execution model, browser storage (`localStorage`, `sessionStorage`, Cookies with `HttpOnly`, `Secure`, `SameSite`).
- **Essential Toolkit Setup:**
  - Interception Proxies: Burp Suite Community/Pro, Caido, or OWASP ZAP.
  - Browser DevTools (Network, Console, Application, Storage tabs).
  - CLI Essentials: `curl`, `jq`, `openssl`, Python scripting.

---

### Phase 2: Reconnaissance & Target Surface Mapping
*Objective: Discover assets, features, and hidden endpoints within authorized scope.*

- **Scope Analysis:**
  - Small scope (single URL) vs Medium scope (wildcard domain) vs Large scope (org-wide CIDR/ASN).
  - Reference: [`01-Reconnaissance/Recon-Target-Based-Strategies.md`](../01-Reconnaissance/Recon-Target-Based-Strategies.md).
- **Search Engine & Code Repository Intelligence:**
  - Google Dorking: Finding sensitive file types, exposed portals, and login pages safely.
    - Reference: [`01-Reconnaissance/Google-Dorking-Safe-Guide.md`](../01-Reconnaissance/Google-Dorking-Safe-Guide.md).
  - GitHub Dorking: Uncovering accidental token leaks, test configurations, and internal endpoints.
    - Reference: [`01-Reconnaissance/GitHub-Dorking-Safe-Guide.md`](../01-Reconnaissance/GitHub-Dorking-Safe-Guide.md).
  - Shodan & Internet-Wide Scanning: Discovering forgotten servers, open management interfaces, and SSL certificate pivots.
    - Reference: [`01-Reconnaissance/Shodan-Dorking-Safe-Guide.md`](../01-Reconnaissance/Shodan-Dorking-Safe-Guide.md).
- **Target Fingerprinting:**
  - Technology stack identification (Wappalyzer, WhatWeb, HTTP response headers).
  - Content discovery via non-intrusive directory mapping and historical URL archives (Wayback Machine, AlienVault OTX).

---

### Phase 3: Authentication & Account Security
*Objective: Master user identity lifecycle, token handling, and account takeover vectors.*

- **Registration & Onboarding:**
  - Testing registration flows for duplicate email collisions, role assignment, and unverified activation.
  - Reference: [`03-Authentication-and-Account-Security/Register-Vulnerability-Testing.md`](../03-Authentication-and-Account-Security/Register-Vulnerability-Testing.md).
- **Authentication Endpoints & APIs:**
  - Form-based logins vs JSON API endpoints (`/api/v1/login`).
  - Reference: [`03-Authentication-and-Account-Security/JSON-Authentication-Endpoint-Testing.md`](../03-Authentication-and-Account-Security/JSON-Authentication-Endpoint-Testing.md) and [`Authentication-Endpoint-Testing.md`](../03-Authentication-and-Account-Security/Authentication-Endpoint-Testing.md).
  - Authentication bypasses (parameter tampering, type confusion, response manipulation).
  - Reference: [`03-Authentication-and-Account-Security/Authentication-Bypass-Security-Testing.md`](../03-Authentication-and-Account-Security/Authentication-Bypass-Security-Testing.md).
- **Two-Factor Authentication (2FA/MFA):**
  - Response tampering, direct request access, OTP brute force, and token lifecycle flaws.
  - Reference: [`03-Authentication-and-Account-Security/2FA-Bypass-Security-Testing.md`](../03-Authentication-and-Account-Security/2FA-Bypass-Security-Testing.md).
- **Password Reset Mechanisms:**
  - Token entropy, host header leakage, referrer leakage, and session fixation.
  - Reference: [`03-Authentication-and-Account-Security/Password-Reset-Security-Checklist.md`](../03-Authentication-and-Account-Security/Password-Reset-Security-Checklist.md).
- **Account Takeover (ATO) Vectors:**
  - Chaining authentication and profile update weaknesses to achieve complete takeover.
  - Reference: [`03-Authentication-and-Account-Security/Account-Takeover-ATO-Security-Testing.md`](../03-Authentication-and-Account-Security/Account-Takeover-ATO-Security-Testing.md).
- **Session & Cookie Security:**
  - Cookie attributes, session termination, session fixation, and token binding.
  - Reference: [`03-Authentication-and-Account-Security/Cookie-Security-Testing.md`](../03-Authentication-and-Account-Security/Cookie-Security-Testing.md).

---

### Phase 4: Core Web & API Vulnerabilities
*Objective: Uncover high-impact, repeatable vulnerability classes in business web applications.*

- **Insecure Direct Object References (IDOR) & Broken Object-Level Authorization (BOLA):**
  - Testing numeric, GUID, and string identifiers across CRUD endpoints with multiple testing roles.
  - Reference: [`02-Web-Vulnerabilities/IDOR-Security-Testing.md`](../02-Web-Vulnerabilities/IDOR-Security-Testing.md), [`IDOR-Testing-Advanced.md`](../02-Web-Vulnerabilities/IDOR-Testing-Advanced.md), and [`Authorization-Testing-IDOR.md`](../02-Web-Vulnerabilities/Authorization-Testing-IDOR.md).
- **Cross-Site Scripting (XSS):**
  - Reflected, Stored, and DOM-based XSS; contexts (HTML, attribute, JavaScript variable, URL).
  - Safe payload design and defensive encoding verification.
  - Reference: [`02-Web-Vulnerabilities/XSS-Testing-Methods.md`](../02-Web-Vulnerabilities/XSS-Testing-Methods.md) and [`XSS-Cheat-Sheet-Basic-and-Advanced.md`](../02-Web-Vulnerabilities/XSS-Cheat-Sheet-Basic-and-Advanced.md).
- **File Upload Vulnerabilities:**
  - Extension validation bypasses, MIME-type trust, polyglot uploads, path traversal in filename.
  - Reference: [`02-Web-Vulnerabilities/File-Upload-Security-Testing.md`](../02-Web-Vulnerabilities/File-Upload-Security-Testing.md).
- **Host Header Injection & Cache Interactions:**
  - Password reset poisoning, cache poisoning, routing redirection.
  - Reference: [`02-Web-Vulnerabilities/Host-Header-Injection-Security-Testing.md`](../02-Web-Vulnerabilities/Host-Header-Injection-Security-Testing.md).
- **Mass Assignment / Parameter Binding:**
  - Injecting unexpected schema properties into JSON or form payloads (e.g., `is_admin`, `verified`).
  - Reference: [`02-Web-Vulnerabilities/Mass-Assignment-Security-Testing.md`](../02-Web-Vulnerabilities/Mass-Assignment-Security-Testing.md).
- **Exposed Source Code & Information Disclosure:**
  - Public `.git`, backup files (`.bak`, `.old`), sourcemaps, and internal stack traces.
  - Reference: [`02-Web-Vulnerabilities/Exposed-Source-Code-Testing.md`](../02-Web-Vulnerabilities/Exposed-Source-Code-Testing.md).

---

### Phase 5: Business Logic, Access Controls & Technology Reviews
*Objective: Look beyond automated scanners by understanding complex workflows and framework specifics.*

- **Business Logic Flaws:**
  - Multi-step checkouts, pricing manipulation, negative quantities, state machine desynchronization.
  - Reference: [`02-Web-Vulnerabilities/Business-Logic-Security-Testing.md`](../02-Web-Vulnerabilities/Business-Logic-Security-Testing.md).
- **Access Control & 403 Forbidden Bypasses:**
  - Path normalization tricks, custom headers (`X-Original-URL`, `X-Rewrite-URL`), method override.
  - Reference: [`02-Web-Vulnerabilities/403-Forbidden-Testing.md`](../02-Web-Vulnerabilities/403-Forbidden-Testing.md) and [`Admin-Panel-Security-Testing.md`](../03-Authentication-and-Account-Security/Admin-Panel-Security-Testing.md).
- **Rate Limiting & Abuse Prevention:**
  - Header spoofing, IP rotation checks, concurrency tests.
  - Reference: [`02-Web-Vulnerabilities/Rate-Limit-Security-Testing.md`](../02-Web-Vulnerabilities/Rate-Limit-Security-Testing.md).
- **Framework & Technology-Specific Audits:**
  - Django: Debug mode, secret key exposure, ORM injection, admin misconfigurations.
    - Reference: [`04-Technology-Specific/Django-Security-Review.md`](../04-Technology-Specific/Django-Security-Review.md).
  - Symfony: Profiler leakage (`_profiler`), secret exposure, debug routes.
    - Reference: [`04-Technology-Specific/Symfony-Security-Review.md`](../04-Technology-Specific/Symfony-Security-Review.md).
  - Atlassian Jira: Public user enumeration, dashboard misconfigurations, unauthenticated REST APIs.
    - Reference: [`04-Technology-Specific/Jira-Security-Review.md`](../04-Technology-Specific/Jira-Security-Review.md).
  - Metadata & Privacy: EXIF data extraction on user media uploads.
    - Reference: [`04-Technology-Specific/EXIF-Metadata-Privacy-Testing.md`](../04-Technology-Specific/EXIF-Metadata-Privacy-Testing.md).

---

### Phase 6: Advanced Research, Chaining & Professional Reporting
*Objective: Build maximum severity findings through chaining and communicate clearly with triage teams.*

- **Remote Code Execution (RCE) Classes:**
  - Command injection, unsafe deserialization, template injection (SSTI), code evaluation.
  - Reference: [`02-Web-Vulnerabilities/Remote-Code-Execution-Classes.md`](../02-Web-Vulnerabilities/Remote-Code-Execution-Classes.md) and [`Command-Injection-Security-Testing.md`](../02-Web-Vulnerabilities/Command-Injection-Security-Testing.md).
- **Vulnerability Chaining:**
  - Low-impact CSRF or self-XSS combined with CORS misconfiguration to yield complete Account Takeover.
  - Open redirect combined with OAuth implementation to leak authorization codes.
- **Field Notes & Research Methodology:**
  - Tips for maintaining stamina, avoiding duplicate reports, and organizing notes.
  - Reference: [`05-Research-and-Field-Notes/Bug-Bounty-Research-Tips.md`](../05-Research-and-Field-Notes/Bug-Bounty-Research-Tips.md) and [`Bug-Bounty-Research-Tips-Part-2.md`](../05-Research-and-Field-Notes/Bug-Bounty-Research-Tips-Part-2.md).
- **Master Checklist Integration:**
  - Comprehensive index of all 27 audit domains.
  - Reference: [`05-Research-and-Field-Notes/Vulnerability-Checklist-Index.md`](../05-Research-and-Field-Notes/Vulnerability-Checklist-Index.md).
- **Professional Reporting & Responsible Disclosure:**
  - Write concise, reproducible proof-of-concepts (PoCs).
  - Calculate realistic CVSS 3.1 / 4.0 severity scores based on actual impact.
  - Follow program guidelines outlined in [`DISCLAIMER.md`](../DISCLAIMER.md).

---

## 📚 Recommended Practice Labs & Platforms

| Platform | Recommended Focus | Difficulty |
| :--- | :--- | :--- |
| **PortSwigger Web Security Academy** | Interactive labs for IDOR, XSS, CSRF, SSRF, SQLi, and Business Logic | Beginner to Advanced |
| **OWASP Juice Shop** | Full-stack modern JavaScript application with realistic vulnerabilities | Beginner to Intermediate |
| **Damn Vulnerable Web Application (DVWA)** | Classic web vulnerabilities with adjustable security levels | Beginner |
| **HackTheBox / TryHackMe** | Full machine compromise, network pivoting, and CTF challenges | Intermediate to Advanced |
| **Bugcrowd University / Hacker101** | Structured modules designed specifically for bug bounty methodologies | All Levels |

---

## 📈 Learning Progress Checklist

- [ ] Complete Phase 1: Tooling and HTTP protocol mastery.
- [ ] Complete Phase 2: Reconnaissance and scope discipline.
- [ ] Complete Phase 3: Authentication and account security testing.
- [ ] Complete Phase 4: Core web vulnerability identification and safe verification.
- [ ] Complete Phase 5: Business logic analysis and framework reviews.
- [ ] Complete Phase 6: Vulnerability chaining and professional vulnerability report writing.
- [ ] Track your 55-topic progress in [`06-Learning-Resources/Bug-Bounty-Advance-Training-Checklist.md`](Bug-Bounty-Advance-Training-Checklist.md).
