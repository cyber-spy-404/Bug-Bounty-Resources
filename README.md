# 🎭 Cyber_Spy.404 | Bug Bounty Reference Library

[![Security](https://img.shields.io/badge/Focus-Bug%20Bounty%20%26%20AppSec-red.svg)](#)
[![Notes Count](https://img.shields.io/badge/Notes-40%20Modules-blue.svg)](#)
[![Categories](https://img.shields.io/badge/Categories-6%20Domains-green.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Ethics](https://img.shields.io/badge/Ethics-Authorized%20Testing%20Only-brightgreen.svg)](DISCLAIMER.md)

Welcome to the **Cyber_Spy.404** educational repository. This library hosts curated security research documentation, application checklists, and testing workflows derived from the *ICC Bug Bounty Notes* series. It serves as a structured, scannable handbook for learning and methodology development.

> 📥 **Download Offline Edition:** Get our consolidated [**Cyber-Spy-404 Complete Bug Bounty Handbook (PDF)**](07-PDF-Downloads/Cyber-Spy-404-Bug-Bounty-Complete-Handbook.pdf) (~188 KB) or explore all 33 individual module guides in [**07-PDF-Downloads**](07-PDF-Downloads/README.md).

---

## 🛑 Security Disclaimer & Ethics
> **Important:** This repository is maintained strictly for educational research, defensive security engineering, and authorized application auditing. Unsanctioned testing, exploitation, or data gathering against unauthorized systems is strictly prohibited. All techniques detailed here must be executed inside isolated staging environments or within the explicitly defined boundaries of an approved bug bounty program.

---

## 🛠️ Testing Environment
Research and validation are conducted across a customized technical testing stack:
* **Host Platform:** Windows Environment (System Administration & Development Stack)
* **Testing Sandbox:** Intel Processor-based MacBook Pro running a customized deployment of macOS Monterey

---

## 💬 Community & Sync
This repository maps 1-to-1 with our private Discord workspace channels. Use the direct paths inside this index to navigate the resource drops posted in the respective channel branches.

* **Author:** Cyber_Spy.404
* **Project Status:** Active Research Phase

---

## 📁 Repository Structure & Directory Index

Navigate directly to specific security branches using the index below:

### 🪟 1. Web Vulnerabilities (`/02-Web-Vulnerabilities`)
* [Injections](02-Web-Vulnerabilities/Command-Injection-Security-Testing.md) — Technical breakdowns of Command, CRLF, and Host Header injection validation paths.
* [XSS & Cookies](02-Web-Vulnerabilities/XSS-Testing-Methods.md) — Non-malicious verification workflows for Cross-Site Scripting.
* [IDOR & Authorization](02-Web-Vulnerabilities/IDOR-Security-Testing.md) — Verification logs for parameter tampering, horizontal privileges, and `403` routing bypasses.
* [CSRF](02-Web-Vulnerabilities/CSRF-Security-Testing-Guide.md) — Token implementation flaws and cross-site interaction analysis.
* [RCE & SSTI](02-Web-Vulnerabilities/Remote-Code-Execution-Classes.md) — Local parsing validation checks for backend engines.
* [Business Logic](02-Web-Vulnerabilities/Business-Logic-Security-Testing.md) — Workflows for auditing multi-factor flows, account registration logic, and password flows.

### 🪟 2. Reconnaissance & Mapping (`/01-Reconnaissance`)
* [Dorking & Engines](01-Reconnaissance/Google-Dorking-Safe-Guide.md) — Scannable indices for Google Dorking structures and passive OSINT asset discovery.
* [Target Recon](01-Reconnaissance/Recon-Target-Based-Strategies.md) — Target surface mapping and metadata examination methodologies.

### 🪟 3. Methodology & Frameworks (`/05-Research-and-Field-Notes`)
* [Audit Checklists](05-Research-and-Field-Notes/Vulnerability-Checklist-Index.md) — General vulnerability assessment baseline lists.
* [File Upload Bypasses](02-Web-Vulnerabilities/File-Upload-Security-Testing.md) — Multi-part request structure checks.
* [Rate Limiting](02-Web-Vulnerabilities/Rate-Limit-Security-Testing.md) — API restriction validations.
* [Identity & Data](03-Authentication-and-Account-Security/) — Authentication structural auditing, JSON parameters, and system misconfiguration testing.
* [Code Review](04-Technology-Specific/) — Security baseline audits for Django, Symfony, and general source disclosure testing.

### 🪟 4. Offline Handbook & PDF Downloads (`/07-PDF-Downloads`)
* [📕 Complete Bug Bounty Handbook (PDF)](07-PDF-Downloads/Cyber-Spy-404-Bug-Bounty-Complete-Handbook.pdf) — All-in-one consolidated multi-chapter offline handbook (~188 KB).
* [📁 Categorized Module PDFs Index](07-PDF-Downloads/README.md) — 33 individual standalone PDF guides organized by security domain.

---

## ⚡ Interactive Web Suite Tools (`/html files`)
Launch any of our local browser-based security apps with dynamic parameter injection, search, and direct hands-on lab links:
* [**Web Suite Dashboard**](html%20files/index.html) — Universal search engine across all 40 guides + Practice Labs Hub.
* [**Google Dorks Explorer**](html%20files/google%20dorks.html) — Dynamic target domain injection & 1-click Google search launcher.
* [**Shodan Intelligence Tool**](html%20files/shodan.html) — Host, Org, ASN filters and automated Shodan query execution.
* [**GitHub Dorks Generator**](html%20files/GitHub%20Dorking.html) — Leaked secrets, token, and API key finder for GitHub Code Search.
* [**Reconnaissance Matrix**](html%20files/Recon%20Target%20Based.html) — Target-based scope planner (Small/Medium/Large) and command generator.
* [**Learning Progress Tracker**](html%20files/Bug%20Bounty%20Advance%20Training.html) — 55-topic syllabus tracker with persistent notes & lab links.

---

## 📑 Complete Table of Contents

### 🔍 01. Reconnaissance
Target surface mapping, scope-dependent discovery strategies, and passive intelligence gathering.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**Recon-Target-Based-Strategies.md**](01-Reconnaissance/Recon-Target-Based-Strategies.md) | Small vs Medium vs Large Scope | Scope classification, parameter fuzzing limits, subdomain takeovers |
| [**Google-Dorking-Safe-Guide.md**](01-Reconnaissance/Google-Dorking-Safe-Guide.md) | Search Engine Intelligence | Finding exposed dashboards, backup files, configuration leaks safely |
| [**GitHub-Dorking-Safe-Guide.md**](01-Reconnaissance/GitHub-Dorking-Safe-Guide.md) | Code Repository Recon | Uncovering exposed developer tokens, staging URLs, sensitive keys |
| [**Shodan-Dorking-Safe-Guide.md**](01-Reconnaissance/Shodan-Dorking-Safe-Guide.md) | Internet-Wide Asset Discovery | SSL certificate pivots, exposed admin panels, unauthenticated services |

---

### 🌐 02. Web Vulnerabilities
Methodology and verification guides for core server-side and client-side web application flaws.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**IDOR-Security-Testing.md**](02-Web-Vulnerabilities/IDOR-Security-Testing.md) | Insecure Direct Object References | Parameter substitution, GUID rotation, CRUD verification, horizontal privilege escalation |
| [**IDOR-Testing-Advanced.md**](02-Web-Vulnerabilities/IDOR-Testing-Advanced.md) | Advanced IDOR & BOLA | Multi-tenant boundaries, parameter pollution, mass export leaks, GraphQL node traversal |
| [**Authorization-Testing-IDOR.md**](02-Web-Vulnerabilities/Authorization-Testing-IDOR.md) | Access Control & Permissions | Vertical/horizontal escalation, role matrices, status code validation, session state isolation |
| [**403-Forbidden-Testing.md**](02-Web-Vulnerabilities/403-Forbidden-Testing.md) | 403 Forbidden Bypasses | Header spoofing (`X-Original-URL`), path traversal, case manipulation, method override |
| [**Business-Logic-Security-Testing.md**](02-Web-Vulnerabilities/Business-Logic-Security-Testing.md) | Business Logic Flaws | Currency tampering, workflow stage skips, race conditions, negative number handling |
| [**Rate-Limit-Security-Testing.md**](02-Web-Vulnerabilities/Rate-Limit-Security-Testing.md) | Throttling & Rate Limiting | Header rotation, path fuzzing, method switching, distributed source requests |
| [**XSS-Testing-Methods.md**](02-Web-Vulnerabilities/XSS-Testing-Methods.md) | Cross-Site Scripting Testing | Context breakout, filter evasion, source-to-sink tracing, CSP evaluation |
| [**XSS-Cheat-Sheet-Basic-and-Advanced.md**](02-Web-Vulnerabilities/XSS-Cheat-Sheet-Basic-and-Advanced.md) | Safe Payloads & Polyglots | HTML attribute breakouts, JavaScript context payloads, event handlers, non-destructive alerts |
| [**File-Upload-Security-Testing.md**](02-Web-Vulnerabilities/File-Upload-Security-Testing.md) | File Upload Vulnerabilities | Extension blacklists, MIME type tampering, magic bytes, SVG stored XSS, double extensions |
| [**Host-Header-Injection-Security-Testing.md**](02-Web-Vulnerabilities/Host-Header-Injection-Security-Testing.md) | Host Header Injection | Password reset poisoning, cache poisoning, routing-based SSRF, forward-header spoofing |
| [**Mass-Assignment-Security-Testing.md**](02-Web-Vulnerabilities/Mass-Assignment-Security-Testing.md) | Mass Assignment / Over-Posting | Parameter addition, role-elevation injection (`is_admin: true`), schema exploration |
| [**Exposed-Source-Code-Testing.md**](02-Web-Vulnerabilities/Exposed-Source-Code-Testing.md) | Source Code & Backup Leaks | `.git/` folder reconstruction, `.env` file discovery, `.bak`/swap files, secret identification |
| [**Remote-Code-Execution-Classes.md**](02-Web-Vulnerabilities/Remote-Code-Execution-Classes.md) | RCE Vulnerability Classes | Command injection, SSTI, unsafe deserialization, file inclusion primitives |
| [**Command-Injection-Security-Testing.md**](02-Web-Vulnerabilities/Command-Injection-Security-Testing.md) | OS Command Injection | Argument separators, blind timing checks, out-of-band detection, shell metacharacter evasion |
| [**CRLF-Injection-Security-Testing.md**](02-Web-Vulnerabilities/CRLF-Injection-Security-Testing.md) | CRLF & HTTP Splitting | Header injection (`\r\nSet-Cookie:`), response splitting, cache deception, redirect hijacking |
| [**CSRF-Security-Testing-Guide.md**](02-Web-Vulnerabilities/CSRF-Security-Testing-Guide.md) | Cross-Site Request Forgery | Token presence checks, method switching (`POST` to `GET`), SameSite attribute interactions |
| [**CSRF-Security-Testing-Methods.md**](02-Web-Vulnerabilities/CSRF-Security-Testing-Methods.md) | Advanced CSRF Verification | Referer header stripping, double-submit cookie flaws, CORS preflight edge cases |
| [**Denial-of-Service-Security-Testing.md**](02-Web-Vulnerabilities/Denial-of-Service-Security-Testing.md) | Application-Layer DoS | ReDoS detection, large payload processing, decompression bombs, safe verification thresholds |

---

### 🔐 03. Authentication and Account Security
Auditing authentication workflows, credential lifecycle, session management, and account protections.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**Register-Vulnerability-Testing.md**](03-Authentication-and-Account-Security/Register-Vulnerability-Testing.md) | Registration Logic | Duplicate registration collisions, role injection, pre-account takeover, parameter pollution |
| [**Authentication-Endpoint-Testing.md**](03-Authentication-and-Account-Security/Authentication-Endpoint-Testing.md) | Login Endpoints | Username enumeration, status code differentials, timing analysis, credential stuffing defenses |
| [**Authentication-Bypass-Security-Testing.md**](03-Authentication-and-Account-Security/Authentication-Bypass-Security-Testing.md) | Authentication Bypasses | Direct URL access, forced browsing, response tampering (`{"auth": true}`), OAuth redirect flaws |
| [**JSON-Authentication-Endpoint-Testing.md**](03-Authentication-and-Account-Security/JSON-Authentication-Endpoint-Testing.md) | API & JSON Auth | Content-Type switching, NoSQL injection (`$ne`), type juggling, JWT signature bypasses |
| [**2FA-Bypass-Security-Testing.md**](03-Authentication-and-Account-Security/2FA-Bypass-Security-Testing.md) | Multi-Factor Authentication | Code brute forcing without throttling, response manipulation, direct route access, token reuse |
| [**Password-Reset-Security-Checklist.md**](03-Authentication-and-Account-Security/Password-Reset-Security-Checklist.md) | Password Reset Workflows | Token predictability, host header reset poisoning, referrer leakage, session invalidation |
| [**Account-Takeover-ATO-Security-Testing.md**](03-Authentication-and-Account-Security/Account-Takeover-ATO-Security-Testing.md) | Account Takeover (ATO) | Vulnerability chaining, email verification race conditions, OAuth implementation flaws |
| [**Admin-Panel-Security-Testing.md**](03-Authentication-and-Account-Security/Admin-Panel-Security-Testing.md) | Administrative Portals | Hidden path discovery, IP whitelist spoofing, credential defaults, cookie manipulation |
| [**Cookie-Security-Testing.md**](03-Authentication-and-Account-Security/Cookie-Security-Testing.md) | Cookies & Session Security | `HttpOnly`, `Secure`, `SameSite` flags, session fixation, token entropy, invalidation on logout |

---

### ⚙️ 04. Technology-Specific
Auditing frameworks, third-party software products, and application asset handlers.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**Django-Security-Review.md**](04-Technology-Specific/Django-Security-Review.md) | Django Framework Security | `DEBUG=True` exposure, `SECRET_KEY` risks, ORM injection, admin protection |
| [**Symfony-Security-Review.md**](04-Technology-Specific/Symfony-Security-Review.md) | Symfony Framework Security | `_profiler` leakage, debug toolbar, environment secrets, routing misconfigurations |
| [**Jira-Security-Review.md**](04-Technology-Specific/Jira-Security-Review.md) | Atlassian Jira Security | User enumeration, public filter & dashboard leaks, unauthenticated REST APIs |
| [**EXIF-Metadata-Privacy-Testing.md**](04-Technology-Specific/EXIF-Metadata-Privacy-Testing.md) | EXIF Metadata & Privacy | GPS location extraction from user uploads, metadata retention testing |

---

### 📝 05. Research and Field Notes
Field-tested tips, mindset, target selection, report triage advice, and comprehensive index checklists.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**Target-Selection-Strategy-Guide.md**](05-Research-and-Field-Notes/Target-Selection-Strategy-Guide.md) | Beginner Target Selection | Solving the target selection crisis, VDP reputation pipeline, high-yield SaaS checklist |
| [**Bug-Bounty-Research-Tips.md**](05-Research-and-Field-Notes/Bug-Bounty-Research-Tips.md) | Hunting Strategy & Workflow | Avoiding duplicate reports, target selection, finding low-hanging fruit |
| [**Bug-Bounty-Research-Tips-Part-2.md**](05-Research-and-Field-Notes/Bug-Bounty-Research-Tips-Part-2.md) | Deep Testing & Methodology | Feature deep dives, reading JavaScript, thinking like an application developer |
| [**Vulnerability-Checklist-Index.md**](05-Research-and-Field-Notes/Vulnerability-Checklist-Index.md) | Master Vulnerability Index | Complete 27-category checklist spanning web, API, and framework security + Practice Labs |

---

### 🎓 06. Learning Resources
Structured education, milestones, training syllabi, and interactive practice tracking.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**Learning-Roadmap.md**](06-Learning-Resources/Learning-Roadmap.md) | 6-Phase Master Roadmap | Beginner to advanced journey, recommended labs, core reading milestones |
| [**Real-World-Practice-Labs-Guide.md**](06-Learning-Resources/Real-World-Practice-Labs-Guide.md) | Hands-on Labs & Motivation | PortSwigger, HTB, THM, Hacker101 CTF, crAPI, Dual-Screen hunting method |
| [**Bug-Bounty-Advance-Training-Checklist.md**](06-Learning-Resources/Bug-Bounty-Advance-Training-Checklist.md) | Interactive Training Tracker | 55-topic progress tracker covering vulnerabilities, CVEs, bypasses + Labs |

---

## 🎯 Recommended Learning Path

```
[Phase 1: Foundations]
   └─ 06-Learning-Resources/Learning-Roadmap.md
   └─ 01-Reconnaissance/Recon-Target-Based-Strategies.md
        │
        ▼
[Phase 2: Authentication & Account Lifecycle]
   └─ 03-Authentication-and-Account-Security/Register-Vulnerability-Testing.md
   └─ 03-Authentication-and-Account-Security/Authentication-Endpoint-Testing.md
   └─ 03-Authentication-and-Account-Security/Password-Reset-Security-Checklist.md
   └─ 03-Authentication-and-Account-Security/2FA-Bypass-Security-Testing.md
        │
        ▼
[Phase 3: Core Authorization & Web Vulnerabilities]
   └─ 02-Web-Vulnerabilities/IDOR-Security-Testing.md
   └─ 02-Web-Vulnerabilities/Authorization-Testing-IDOR.md
   └─ 02-Web-Vulnerabilities/XSS-Testing-Methods.md
   └─ 02-Web-Vulnerabilities/File-Upload-Security-Testing.md
   └─ 02-Web-Vulnerabilities/Mass-Assignment-Security-Testing.md
        │
        ▼
[Phase 4: Logic Flaws & Framework Specifics]
   └─ 02-Web-Vulnerabilities/Business-Logic-Security-Testing.md
   └─ 02-Web-Vulnerabilities/403-Forbidden-Testing.md
   └─ 04-Technology-Specific/Django-Security-Review.md / Symfony-Security-Review.md
        │
        ▼
[Phase 5: Field Notes & Mastery]
   └─ 05-Research-and-Field-Notes/Target-Selection-Strategy-Guide.md
   └─ 05-Research-and-Field-Notes/Bug-Bounty-Research-Tips.md
   └─ 06-Learning-Resources/Real-World-Practice-Labs-Guide.md
```

---

## ⚖️ Legal Disclaimer & Ethics

All materials in this repository are for **authorized security testing and educational research only**. Never execute these techniques against targets without explicit, written permission from the asset owner.

Please read our full [**DISCLAIMER.md**](DISCLAIMER.md) before using these notes.

---

## 🤝 Contributing

Contributions, updates, and improvements are welcome! Please review [**CONTRIBUTING.md**](CONTRIBUTING.md) for formatting guidelines, payload safety rules, and pull request procedures.

---

## 📄 License

This documentation is released under the [**MIT License**](LICENSE). Feel free to adapt and share with proper attribution.
