# 📁 Bug Bounty Learning Checklist — Progress Tracker

*Contributed by Cyber_Spy.404 | Bug Bounty Notes*

## 📌 Executive Summary
The source HTML, *Bug Bounty Advance Training.html*, is an interactive checklist for tracking learning progress. It groups 55 topics into vulnerabilities, bypass techniques, functionality testing, CVEs, miscellaneous topics, technologies, and reconnaissance. The original UI's counters initialize at zero and are intended to update as boxes are checked.

## 🔍 Core Testing Methodology
1. Use this checklist as a learning tracker, not as permission to test a live system.
2. Study the concept and its defensive controls before attempting a lab.
3. Practice in deliberately vulnerable labs or on assets covered by written authorization.
4. Keep notes on prerequisites, test design, expected behavior, evidence, and remediation.
5. Mark a topic complete only when you can explain it and demonstrate it safely.
6. For CVEs, verify the affected product/version and consult vendor advisories rather than assuming every listed year applies.

## 💡 High-Value Bug Bounty Tips & Edge Cases

### 🎯 Vulnerabilities (23 items in the HTML)
- [ ] Arbitrary File Upload — 🧪 [PortSwigger File Upload Labs](https://portswigger.net/web-security/file-upload) \| [THM Upload Vulnerabilities](https://tryhackme.com/room/uploadvulnerabilities)
- [ ] BugBounty Programs Dorking — 🧪 [TryHackMe Google Dorking Room](https://tryhackme.com/room/googledorking)
- [ ] CRLF Injection — 🧪 [PentesterLab HTTP Response Splitting](https://pentesterlab.com/) \| [THM Header Flaws](https://tryhackme.com/room/httprequestsmuggling)
- [ ] Cross Site Request Forgery (CSRF) — 🧪 [PortSwigger CSRF Vulnerability Labs](https://portswigger.net/web-security/csrf)
- [ ] Cross Site Scripting (XSS) — 🧪 [PortSwigger XSS Academy Labs](https://portswigger.net/web-security/cross-site-scripting) \| [THM XSS Room](https://tryhackme.com/room/xss)
- [ ] Denial of Service (DoS) — 🧪 [PortSwigger Server-Side Prototype Pollution & DoS](https://portswigger.net/web-security/prototype-pollution/server-side)
- [ ] Exposed Source Code — 🧪 [HTB Machine: Pilgrimage](https://app.hackthebox.com/machines/Pilgrimage) \| [THM Git Happens](https://tryhackme.com/room/githappens)
- [ ] Fast Automation — 🧪 [TryHackMe FFuF Room](https://tryhackme.com/room/ffuf) \| [HTB Academy: Web Fuzzing](https://academy.hackthebox.com/)
- [ ] Host Header Injection — 🧪 [PortSwigger Host Header Attacks](https://portswigger.net/web-security/host-header)
- [ ] Insecure Direct Object References (IDOR) — 🧪 [PortSwigger IDOR Labs](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter) \| [THM IDOR](https://tryhackme.com/room/idor)
- [ ] Local File Inclusion (LFI) — 🧪 [PortSwigger Directory Traversal Labs](https://portswigger.net/web-security/file-path-traversal) \| [THM File Inclusion](https://tryhackme.com/room/fileinc)
- [ ] Mass Assignment — 🧪 [PortSwigger API Mass Assignment Labs](https://portswigger.net/web-security/api-testing/lab-bypassing-access-controls-via-mass-assignment) \| [OWASP crAPI](https://github.com/OWASP/crAPI)
- [ ] NoSQL Injection (NoSQLi) — 🧪 [PortSwigger NoSQL Injection Labs](https://portswigger.net/web-security/nosql-injection) \| [HTB Machine: Shoppy](https://app.hackthebox.com/machines/Shoppy)
- [ ] OAuth Misconfiguration — 🧪 [PortSwigger OAuth 2.0 Authentication Labs](https://portswigger.net/web-security/oauth)
- [ ] Open Redirect — 🧪 [PortSwigger OAuth Account Hijacking via Open Redirect](https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri)
- [ ] Reflected File Download (RFD) — 🧪 [OWASP Reflected File Download Testing](https://owasp.org/)
- [ ] Remote File Inclusion (RFI) — 🧪 [TryHackMe File Inclusion (RFI)](https://tryhackme.com/room/fileinc)
- [ ] Server Side Include Injection (SSI Injection) — 🧪 [PentesterLab SSI Injection Badge](https://pentesterlab.com/)
- [ ] Server Side Request Forgery (SSRF) — 🧪 [PortSwigger SSRF Labs](https://portswigger.net/web-security/ssrf) \| [HTB Machine: Sau](https://app.hackthebox.com/machines/Sau)
- [ ] SQL Injection (SQLi) — 🧪 [PortSwigger SQL Injection Labs](https://portswigger.net/web-security/sql-injection) \| [THM SQL Injection](https://tryhackme.com/room/sqlinjectionlm)
- [ ] Web Cache Deception — 🧪 [PortSwigger Web Cache Deception Labs](https://portswigger.net/web-security/web-cache-poisoning)
- [ ] Web Cache Poisoning — 🧪 [PortSwigger Web Cache Poisoning Labs](https://portswigger.net/web-security/web-cache-poisoning)
- [ ] WordPress Vulnerabilities — 🧪 [TryHackMe WordPress: CVE-2021-29447](https://tryhackme.com/room/wordpresscve202129447)

### 🛡️ Bypass Techniques (4 items in the HTML)
- [ ] Bypass 2FA (Two-Factor Authentication) — 🧪 [PortSwigger 2FA Broken Logic Labs](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)
- [ ] Bypass 403 Forbidden — 🧪 [PortSwigger URL-based Access Control Bypasses](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)
- [ ] Bypass 429 Too Many Requests — 🧪 [PortSwigger Rate Limit & IP Rotation Labs](https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block)
- [ ] Bypass Captcha — 🧪 [OWASP Juice Shop CAPTCHA Bypass](https://owasp.org/www-project-juice-shop/)

> Study bypass categories in labs or under explicit program rules. Do not evade rate limits, access controls, or CAPTCHA on real services without authorization.

### ✅ Functionality Testing (2 items)
- [ ] Forgot Password Functionality — 🧪 [PortSwigger Password Reset Poisoning](https://portswigger.net/web-security/host-header/lab-host-header-basic-password-reset-poisoning)
- [ ] Register Functionality — 🧪 [OWASP Juice Shop User Registration Bypasses](https://owasp.org/www-project-juice-shop/)

### 🔍 CVEs (3 items)
- [ ] CVEs 2021 (e.g. Log4j / Confluence) — 🧪 [TryHackMe Solar: Log4j Room](https://tryhackme.com/room/solar)
- [ ] CVEs 2022 (e.g. Spring4Shell) — 🧪 [TryHackMe Spring4Shell Room](https://tryhackme.com/room/spring4shell)
- [ ] CVEs 2023 (e.g. PaperCut / MOVEit) — 🧪 [Hack The Box CVE Sandboxes](https://app.hackthebox.com/)

### 🎭 Miscellaneous Topics (7 items)
- [ ] Account Takeover — 🧪 [PortSwigger Password Reset Broken Logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)
- [ ] Broken Link Hijacking — 🧪 [TryHackMe Subdomain & Link Hijacking](https://tryhackme.com/)
- [ ] Business Logic Errors — 🧪 [PortSwigger Business Logic Labs](https://portswigger.net/web-security/logic-flaws) \| [Hacker101 CTF Petshop Pro](https://ctf.hacker101.com/)
- [ ] Default Credentials — 🧪 [HTB Machine: BoardLight](https://app.hackthebox.com/machines/BoardLight)
- [ ] Email Spoofing — 🧪 [TryHackMe SPF, DKIM, and DMARC Verification](https://tryhackme.com/)
- [ ] JWT Vulnerabilities — 🧪 [PortSwigger JWT Attacks Labs](https://portswigger.net/web-security/jwt)
- [ ] Tabnabbing — 🧪 [OWASP Reverse Tabnabbing Testing](https://owasp.org/)

### 💻 Technologies (12 items)
- [ ] Apache (HTTP Server) — 🧪 [TryHackMe Apache HTTP Misconfiguration](https://tryhackme.com/)
- [ ] Confluence — 🧪 [TryHackMe Confluence CVE-2022-26134 RCE](https://tryhackme.com/)
- [ ] Grafana — 🧪 [TryHackMe Grafana Directory Traversal](https://tryhackme.com/)
- [ ] HAProxy — 🧪 [PortSwigger HTTP Request Smuggling via Reverse Proxy](https://portswigger.net/web-security/request-smuggling)
- [ ] Jenkins — 🧪 [TryHackMe Jenkins Server Takeover](https://tryhackme.com/)
- [ ] Jira — 🧪 [TryHackMe Jira Auditing Room](https://tryhackme.com/room/jira)
- [ ] Joomla — 🧪 [TryHackMe Joomla CMS Penetration Testing](https://tryhackme.com/)
- [ ] Laravel — 🧪 [Hack The Box Laravel Debug Mode RCE](https://app.hackthebox.com/)
- [ ] Moodle — 🧪 [TryHackMe Moodle LMS Vulnerability Lab](https://tryhackme.com/)
- [ ] Nginx — 🧪 [PortSwigger Host Header Routing SSRF in Nginx](https://portswigger.net/web-security/host-header)
- [ ] WordPress — 🧪 [TryHackMe WordPress Security Testing](https://tryhackme.com/)
- [ ] Zend — 🧪 [PentesterLab PHP Framework Vulnerabilities](https://pentesterlab.com/)

### 🔍 Reconnaissance (4 items)
- [ ] Scope Based Recon — 🧪 [TryHackMe Passive Recon Room](https://tryhackme.com/room/passiverecon)
- [ ] Github Dorks — 🧪 [TryHackMe Git Happens](https://tryhackme.com/room/githappens)
- [ ] Google Dorks — 🧪 [TryHackMe Google Dorking Room](https://tryhackme.com/room/googledorking)
- [ ] Shodan Dorks — 🧪 [TryHackMe Shodan Room](https://tryhackme.com/room/shodan)

## 🛠️ Educational Checklist & Safe Examples
For each topic, record:
```text
Topic: [Checklist item]
Learning resource: [Course / Documentation / Lab]
Lab or authorized target: [Name]
Expected behavior: [Summary]
Observed behavior: [Summary]
Defensive fix: [Summary]
Status: [Not Started / Learning / Practiced / Reviewed]
```

- [ ] Explain the vulnerability in your own words
- [ ] Reproduce only in a lab or authorized scope
- [ ] Identify likely impact and affected security boundary
- [ ] Explain the mitigation
- [ ] Save sanitized notes and links to reliable references

## 🛡️ Mitigation & Remediation
Use the appropriate secure-development practice for the confirmed issue: strong authorization, input validation, safe output encoding, parameterized database queries, secure file handling, rate limiting, robust authentication, secure token handling, patched dependencies, and regression tests.

## 📚 Source Notes
Based on *Bug Bounty Advance Training.html*. The HTML has 23 vulnerability entries, 4 bypass entries, 2 functionality entries, 3 CVE-year entries, 7 miscellaneous entries, 12 technologies, and 4 reconnaissance entries (55 total). The original interface's displayed “0/22” for vulnerabilities conflicts with its 23 listed items; this Markdown follows the actual item list and calls out the mismatch rather than silently changing the source.
