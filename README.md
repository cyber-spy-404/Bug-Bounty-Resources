# 🛡️ Cyber_Spy.404 — Bug Bounty Notes & Security Testing Field Guide

[![Security](https://img.shields.io/badge/Focus-Bug%20Bounty%20%26%20AppSec-red.svg)](#)
[![Notes Count](https://img.shields.io/badge/Notes-40%20Modules-blue.svg)](#)
[![Categories](https://img.shields.io/badge/Categories-6%20Domains-green.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Ethics](https://img.shields.io/badge/Ethics-Authorized%20Testing%20Only-brightgreen.svg)](DISCLAIMER.md)

> A structured, field-tested repository of notes, testing methodologies, checklists, and safe edge-case techniques for bug bounty hunters and application security engineers. Contributed by **Cyber_Spy.404**.

---

## 📌 Repository Overview

This repository organizes application security testing notes into a clean, modular hierarchy designed for rapid reference during active, authorized engagements and systematic skill building.

Every module emphasizes:
- **Scope discipline:** Adhering strictly to program rules without expanding into out-of-scope assets.
- **Safe proof-of-concepts:** Controlled verification using dedicated test accounts without destructive actions.
- **Root-cause understanding:** Exploring how flaws emerge, how developers attempt to patch them, and subtle bypass edge cases.
- **Remediation guidance:** Clear, defensive recommendations to help engineering teams remediate vulnerabilities.

---

## 📂 Repository Architecture

```text
Cyber_Spy.404-Bug-Bounty-Notes/
├── README.md                                  # Repository overview, master index & learning paths
├── CONTRIBUTING.md                            # Contribution and note-formatting guidelines
├── LICENSE                                    # Open source license (MIT with educational notice)
├── DISCLAIMER.md                              # Legal disclaimer & responsible disclosure terms
│
├── 01-Reconnaissance/                         # Target mapping, scope strategies, and safe dorking
├── 02-Web-Vulnerabilities/                    # Core web application & API vulnerability guides
├── 03-Authentication-and-Account-Security/    # Auth, session lifecycle, 2FA/MFA, and ATO prevention
├── 04-Technology-Specific/                    # Platform & framework reviews (Django, Symfony, Jira, EXIF)
├── 05-Research-and-Field-Notes/               # Real-world field notes, research workflow, and master index
└── 06-Learning-Resources/                     # Structured 6-phase learning roadmap & progress checklist
```

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
| [**IDOR-Security-Testing.md**](02-Web-Vulnerabilities/IDOR-Security-Testing.md) | Insecure Direct Object References | CRUD testing with dual test accounts, numeric & GUID patterns |
| [**IDOR-Testing-Advanced.md**](02-Web-Vulnerabilities/IDOR-Testing-Advanced.md) | Advanced IDOR & BOLA | Hashed IDs, HTTP method swapping, multi-tenant boundaries |
| [**Authorization-Testing-IDOR.md**](02-Web-Vulnerabilities/Authorization-Testing-IDOR.md) | Object-Level Access Control | Parameter swapping, API object validation, horizontal & vertical privilege |
| [**403-Forbidden-Testing.md**](02-Web-Vulnerabilities/403-Forbidden-Testing.md) | Access Control & 403 Bypasses | Header injection (`X-Original-URL`), path normalization, HTTP methods |
| [**Business-Logic-Security-Testing.md**](02-Web-Vulnerabilities/Business-Logic-Security-Testing.md) | Business Logic Flaws | Multi-step workflows, pricing tampering, race conditions, edge cases |
| [**Rate-Limit-Security-Testing.md**](02-Web-Vulnerabilities/Rate-Limit-Security-Testing.md) | Rate Limiting & Throttling | Header spoofing, IP rotation checks, concurrency tests |
| [**XSS-Testing-Methods.md**](02-Web-Vulnerabilities/XSS-Testing-Methods.md) | Cross-Site Scripting (XSS) | Reflected, Stored, and DOM XSS methodology, context-based reflection |
| [**XSS-Cheat-Sheet-Basic-and-Advanced.md**](02-Web-Vulnerabilities/XSS-Cheat-Sheet-Basic-and-Advanced.md) | XSS Payloads & Contexts | HTML context, attribute context, script block, SVG/event handlers |
| [**File-Upload-Security-Testing.md**](02-Web-Vulnerabilities/File-Upload-Security-Testing.md) | Arbitrary File Uploads | Extension blacklist/whitelist bypasses, MIME-type trust, polyglots |
| [**Host-Header-Injection-Security-Testing.md**](02-Web-Vulnerabilities/Host-Header-Injection-Security-Testing.md) | Host Header Attacks | Password reset poisoning, cache poisoning, routing redirection |
| [**Mass-Assignment-Security-Testing.md**](02-Web-Vulnerabilities/Mass-Assignment-Security-Testing.md) | Mass Assignment / Auto-Binding | Schema injection, role escalation parameters (`is_admin`, `role`) |
| [**Exposed-Source-Code-Testing.md**](02-Web-Vulnerabilities/Exposed-Source-Code-Testing.md) | Information Disclosure | Public `.git`, backup files (`.bak`, `.old`), sourcemap analysis |
| [**Remote-Code-Execution-Classes.md**](02-Web-Vulnerabilities/Remote-Code-Execution-Classes.md) | RCE Vulnerability Classes | Unsafe deserialization, SSTI, file inclusion, code evaluation |
| [**Command-Injection-Security-Testing.md**](02-Web-Vulnerabilities/Command-Injection-Security-Testing.md) | OS Command Injection | Argument separators, blind command injection with delays, escapes |
| [**CRLF-Injection-Security-Testing.md**](02-Web-Vulnerabilities/CRLF-Injection-Security-Testing.md) | CRLF & Response Splitting | Header injection (`%0d%0a`), cookie injection, log poisoning |
| [**CSRF-Security-Testing-Guide.md**](02-Web-Vulnerabilities/CSRF-Security-Testing-Guide.md) | CSRF Fundamentals | Token presence, state-changing GET, `SameSite` cookie interactions |
| [**CSRF-Security-Testing-Methods.md**](02-Web-Vulnerabilities/CSRF-Security-Testing-Methods.md) | Advanced CSRF Testing | Header-based origin validation, token stripping, JSON-based CSRF |
| [**Denial-of-Service-Security-Testing.md**](02-Web-Vulnerabilities/Denial-of-Service-Security-Testing.md) | Application-Level DoS | ReDoS, large payload handling, hash collisions, safe verification |

---

### 🔑 03. Authentication and Account Security
Identification, authentication protocols, session state management, and account takeover vectors.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**Register-Vulnerability-Testing.md**](03-Authentication-and-Account-Security/Register-Vulnerability-Testing.md) | User Registration Flows | Duplicate email collision, unverified account pre-creation, role parameters |
| [**Authentication-Endpoint-Testing.md**](03-Authentication-and-Account-Security/Authentication-Endpoint-Testing.md) | Login & Authentication APIs | Username enumeration, credential handling, error response discrepancies |
| [**Authentication-Bypass-Security-Testing.md**](03-Authentication-and-Account-Security/Authentication-Bypass-Security-Testing.md) | Authentication Bypasses | Direct object requests, status code manipulation, type juggling |
| [**JSON-Authentication-Endpoint-Testing.md**](03-Authentication-and-Account-Security/JSON-Authentication-Endpoint-Testing.md) | JSON Authentication | Content-Type tampering, SQLi/NoSQLi in JSON payloads, type confusion |
| [**2FA-Bypass-Security-Testing.md**](03-Authentication-and-Account-Security/2FA-Bypass-Security-Testing.md) | Two-Factor Authentication (2FA) | Direct endpoint access, OTP brute force, response tampering, backup codes |
| [**Password-Reset-Security-Checklist.md**](03-Authentication-and-Account-Security/Password-Reset-Security-Checklist.md) | Password Reset Mechanisms | Token entropy, Host header poisoning, referrer leakage, token reuse |
| [**Account-Takeover-ATO-Security-Testing.md**](03-Authentication-and-Account-Security/Account-Takeover-ATO-Security-Testing.md) | Account Takeover (ATO) | Chaining auth flaws, OAuth implementation errors, session fixation |
| [**Admin-Panel-Security-Testing.md**](03-Authentication-and-Account-Security/Admin-Panel-Security-Testing.md) | Admin Interfaces & Portals | Path discovery, IP restriction bypasses, role verification on routes |
| [**Cookie-Security-Testing.md**](03-Authentication-and-Account-Security/Cookie-Security-Testing.md) | Cookies & Session State | `HttpOnly`, `Secure`, `SameSite` flags, token expiration, session revival |

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
Field-tested tips, mindset, report triage advice, and comprehensive index checklists.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**Target-Selection-Strategy-Guide.md**](05-Research-and-Field-Notes/Target-Selection-Strategy-Guide.md) | Beginner Target Selection | Solving the target selection crisis, VDP reputation pipeline, high-yield SaaS checklist |
| [**Bug-Bounty-Research-Tips.md**](05-Research-and-Field-Notes/Bug-Bounty-Research-Tips.md) | Hunting Strategy & Workflow | Avoiding duplicate reports, target selection, finding low-hanging fruit |
| [**Bug-Bounty-Research-Tips-Part-2.md**](05-Research-and-Field-Notes/Bug-Bounty-Research-Tips-Part-2.md) | Deep Testing & Methodology | Feature deep dives, reading JavaScript, thinking like an application developer |
| [**Vulnerability-Checklist-Index.md**](05-Research-and-Field-Notes/Vulnerability-Checklist-Index.md) | Master Vulnerability Index | Complete 27-category checklist spanning web, API, and framework security |

---

### 🎓 06. Learning Resources
Structured education, milestones, training syllabi, and interactive practice tracking.

| Note | Focus Area | Key Techniques |
| :--- | :--- | :--- |
| [**Learning-Roadmap.md**](06-Learning-Resources/Learning-Roadmap.md) | 6-Phase Master Roadmap | Beginner to advanced journey, recommended labs, core reading milestones |
| [**Real-World-Practice-Labs-Guide.md**](06-Learning-Resources/Real-World-Practice-Labs-Guide.md) | Hands-on Labs & Motivation | PortSwigger, HTB, THM, Hacker101 CTF, crAPI, Dual-Screen hunting method |
| [**Bug-Bounty-Advance-Training-Checklist.md**](06-Learning-Resources/Bug-Bounty-Advance-Training-Checklist.md) | Interactive Training Tracker | 55-topic progress tracker covering vulnerabilities, CVEs, and bypasses |

---

## 🎯 Recommended Learning Path

For researchers progressing through these materials, we recommend the following sequence:

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
   └─ 05-Research-and-Field-Notes/Bug-Bounty-Research-Tips.md
   └─ 06-Learning-Resources/Bug-Bounty-Advance-Training-Checklist.md
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
