# 🧪 Real-World Practice Labs & Motivation Guide

*Contributed by Cyber_Spy.404 | Bug Bounty Knowledge Base*

> **Ethics and Safe Practice:** All techniques discussed below should first be practiced within legal, controlled lab environments (PortSwigger, Hack The Box, TryHackMe, OWASP, Hacker101). Never execute exploits on live infrastructure without explicit, written authorization.

---

## 📌 1. Executive Overview & The Motivation Gap

A major hurdle for aspiring bug hunters is the **"Lab-to-Target Disconnect"**:
1. In a lab, you know a vulnerability is present, giving you false confidence.
2. On a live target, 98% of endpoints are properly configured, which causes hesitation, self-doubt, and burnout.

```
+-----------------------------------------------------------------------------+
|                        THE SKILL TRANSLATION FUNNEL                         |
+-----------------------------------------------------------------------------+
   [Tier 1: Foundations]        PortSwigger Academy + TryHackMe (Web)
             │                  Builds repeatable muscle memory in Burp Suite
             ▼
   [Tier 2: Real Scenarios]     HTB Academy (CBBH) + OWASP crAPI / Juice Shop
             │                  Simulates real full-stack multi-user apps
             ▼
   [Tier 3: Platform CTFs]      Hacker101 CTF (Unlocks HackerOne Private Invites!)
             │                  Direct monetary & invite motivation
             ▼
   [Tier 4: Live Target Hunting] VDPs & Scope Expansion Monitoring
                                First triaged valid finding & payout
```

The key to overcoming this disconnect is **hunting with intent**: practicing a single vulnerability class in a lab in the morning, and looking for that exact same pattern in a live web application in the afternoon.

---

## 🚀 2. Tier 1: Beginner Labs (Foundations & Quick Wins)

These labs are **100% free**, run directly in the browser with proxy integration, and teach the foundational vulnerability classes that account for 60%+ of beginner bounties.

### A. PortSwigger Web Security Academy (The Gold Standard)
*Cost: Free | Setup: Burp Suite Community / Professional*

| Topic Track | Must-Solve Labs | Direct Bug Bounty Translation |
| :--- | :--- | :--- |
| **Authentication** | 1. Username enumeration via different responses<br>2. 2FA simple bypass<br>3. Password reset broken logic | Broken authentication, credential stuffing, password reset takeover on SaaS portals. |
| **Access Control (IDOR)** | 1. Unprotected admin functionality<br>2. User ID controlled by request parameter<br>3. IDOR with unpredictable IDs | Insecure Direct Object References (BOLA), reading other users' receipts, PII leakage. |
| **Business Logic** | 1. Excessive trust in client-side controls<br>2. High-level logic vulnerability<br>3. Inconsistent handling of exceptional input | Cart price tampering, coupon reuse, negative balance manipulation. |
| **File Upload** | 1. Remote code execution via web shell upload<br>2. Flawed validation of file extension<br>3. Insufficient blacklist bypass | Stored XSS via SVG avatar, bypassing image validation to upload `.php` or `.phtml`. |
| **Cross-Site Scripting (XSS)** | 1. Reflected XSS into HTML context with nothing encoded<br>2. Stored XSS into HTML context<br>3. DOM XSS in `document.write` sink | Account takeover, session hijacking, phishing overlays on customer dashboards. |

---

### B. TryHackMe (THM) - Web Fundamentals Pathway
*Cost: Free & Low-Cost Subscription | Format: Guided Browser Rooms*

TryHackMe provides step-by-step guidance for beginners who want explanations before jumping into raw exploitation.

| Room Name | Key Learning Outcomes | Real-World Application |
| :--- | :--- | :--- |
| **Web Fundamentals** | HTTP request/response inspection, cookies, headers, DevTools | Understanding how modern web browsers interact with backends. |
| **Burp Suite: The Basics & Repeater** | Intercepting, modifying, and replaying HTTP requests | The core workflow of 95% of web bug bounty testing. |
| **OWASP Top 10** | Comprehensive walkthrough of the top web vulnerability categories | Broad familiarity with modern security weaknesses. |
| **Upload Vulnerabilities** | MIME-type tampering, extension blacklists, magic bytes | Bypassing profile picture and document upload filters. |
| **IDOR Room** | Parameter tampering, GUID/UUID patterns, horizontal privilege escalation | Finding authorization flaws in multi-tenant SaaS tools. |

---

### C. OWASP crAPI (Completely Ridiculous API)
*Cost: Free (Open Source Docker) | Format: Realistic Single-Page App + Microservices*

crAPI simulates a modern vehicle management service built with React, Python, Java, and NodeJS microservices.
- **Why it motivates**: Unlike synthetic CTF challenges, crAPI feels like a modern SaaS startup.
- **Top Flaws to Practice**:
  - BOLA / IDOR on vehicle location endpoints.
  - Mass assignment on user profile creation (elevating to mechanic/admin).
  - Rate limiting failures on OTP password reset endpoints.
  - Secret keys and JWT signing secrets leaked in frontend bundles.

---

## 🎯 3. Tier 2: Intermediate Labs (Architecture & Real Logic)

Once you can solve basic PortSwigger labs, move to multi-step challenges that require thinking like a software architect.

### A. Hack The Box (HTB) & HTB Academy
*Platform: Hack The Box | Focus: Real Machine Infrastructure & Web Apps*

HTB hosts realistic web applications and entire vulnerable machines designed to mimic production corporate environments.

#### Recommended HTB Academy Bug Bounty Path (CBBH):
- **Web Requests & Attacks**: Deep dive into parameter pollution, header injection, and HTTP tampering.
- **Server-Side Attacks**: Server-Side Template Injection (SSTI), SSRF, and blind command execution.
- **Login Brute Forcing & Rate Limiting**: Header evasion techniques (`X-Forwarded-For`) and session handling.

#### Top Web-Centric HTB Active & Retired Machines:
1. **Shoppy**: NoSQL Injection in authentication + internal subdomain enumeration.
2. **Precious**: Command injection via vulnerable PDF generation library (`pdfkit`).
3. **Sau**: Request-baskets SSRF vulnerability leading to internal service RCE.
4. **BoardLight**: Default CMS credentials leading to internal CRM exploitation and local privilege escalation.

---

### B. Hacker101 CTF (Direct Path to Private Invites!)
*Platform: HackerOne | Cost: 100% Free*

This is the most **financially motivating** lab platform in cybersecurity.
- **The Incentive**: When you earn **26+ points** on Hacker101 CTF, HackerOne automatically puts you into the priority queue for **Private Bug Bounty Program Invitations**!
- **Recommended Challenges**:
  - *Micro-CMS v1 & v2*: Basic Markdown XSS, authentication bypass, SQL injection.
  - *Petshop Pro*: Business logic flaw in checkout process and balance deduction.
  - *Photo Gallery*: Path traversal and SQL injection in image fetching endpoints.
  - *Cody's First Blog*: PHP file inclusion (LFI) and code evaluation.

---

## 🔥 4. Tier 3: Advanced Labs (Chaining & Modern Attack Vectors)

Advanced labs mirror the top-tier reports that earn $5,000 to $20,000+ bounties on HackerOne and Bugcrowd.

### A. PortSwigger Academy (Practitioner & Expert Labs)
- **HTTP Request Smuggling**: Desynchronizing frontend reverse proxies (Cloudflare, Nginx, AWS ALB) from backend servers (CL.TE, TE.CL, and HTTP/2 downgrade attacks).
- **Web Cache Poisoning & Cache Deception**: Exploiting unkeyed HTTP headers to poison edge CDN caches (Akamai, Fastly, Cloudflare) and deliver malicious JavaScript to legitimate users.
- **OAuth 2.0 & OpenID Connect Flaws**: Leaking authorization codes via unvalidated redirect URIs, CSRF in OAuth flows, and pre-account takeover.
- **Race Conditions & Limit Overruns**: Exploiting multi-threaded concurrency flaws in coupon redemption, gift card spending, and voting systems.

---

### B. Intigriti Monthly Community Challenges
*Platform: Intigriti | Format: Monthly Web Security Puzzles*
- Created by top community researchers.
- Often based on real vulnerabilities found in production during active bounty assessments.
- Teaches modern bypasses: DOM clobbering, CSP bypasses, prototype pollution, and parser differentials.

---

### C. PentesterLab (Pro Badges)
*Platform: PentesterLab | Format: Hands-on vulnerable code exercises*
- Excellent for understanding **SAML vulnerabilities**, JWT algorithm confusion (`none` algorithm, RSA-to-HMAC key confusion), and deserialization in Java/Ruby/Python.

---

## 💡 5. The "Lab-to-Bounty Bridge" Strategy

To turn lab practice into real payouts, use the **Dual-Screen Method**:

```
+--------------------------------------+  +--------------------------------------+
|       SCREEN 1: THE LAB WORK         |  |      SCREEN 2: THE REAL TARGET       |
|                                      |  |                                      |
| Solve PortSwigger Lab:               |  | Inspect Authorized Bounty SaaS:      |
| "User ID controlled by request param"|  | Look for the exact same pattern:     |
| URL: /api/users/profile?id=128       |  | URL: /api/v1/workspaces/billing?id=45|
|                                      |  |                                      |
| Test parameter replacement in Burp   |  | Test dual-account swap in Burp       |
+--------------------------------------+  +--------------------------------------+
```

### The 3 Rules for Real-World Finding Success:
1. **Never Hunt Cold**: Warm up your brain by solving 1 PortSwigger lab or reviewing 1 Hacktivity report before you start hunting on a live program.
2. **Focus on One Bug Class per Week**:
   - *Week 1*: Only look for IDORs and Access Control bypasses across all your targets.
   - *Week 2*: Only test File Upload forms and avatar processing.
   - *Week 3*: Only test Password Reset flows and 2FA bypasses.
   - *Week 4*: Only look for Business Logic and pricing errors.
3. **Log Your Experiments**: Treat hunting like a scientist. Keep track of:
   - What endpoint you tested.
   - What parameter you tampered with.
   - What response code came back.
   - Why it succeeded or failed.

---

## 🏆 6. Recommended 90-Day Practice Milestone Roadmap

```
MONTH 1: Foundation Building (30 Days)
├── PortSwigger: Complete 30 Beginner Labs (Auth, IDOR, XSS, CSRF, File Upload)
├── TryHackMe: Complete "Web Fundamentals" and "OWASP Top 10" pathways
└── Milestone: Understand every tab and feature in Burp Suite / Caido

MONTH 2: The Hacker101 Bridge (60 Days)
├── Hacker101 CTF: Earn 26+ points to unlock HackerOne private invites
├── PortSwigger: Complete 20 Practitioner Labs (SSRF, Race Conditions, Business Logic)
├── Deploy OWASP crAPI locally in Docker and find all 10 API vulnerabilities
└── Milestone: Receive your first batch of private program invitations

MONTH 3: Live Hunting & First Payout (90 Days)
├── Target: Join 2 newly launched or active VDP programs with wide scope
├── Methodology: Apply the Dual-Screen Method focusing on IDOR & Account Security
├── Reporting: Write professional, reproducible reports using our CONTRIBUTING.md template
└── Milestone: Submit your first triaged valid vulnerability!
```
