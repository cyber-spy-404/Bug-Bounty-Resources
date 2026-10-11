# 🤝 Contributing to Cyber_Spy.404 Bug Bounty Notes

Thank you for your interest in contributing to the **Cyber_Spy.404 Bug Bounty Notes** repository! This project aims to be a high-quality, practical, and ethically grounded reference for bug bounty hunters and application security practitioners.

To maintain clarity, readability, and consistency across all topics, please follow the guidelines below.

---

## 📋 Naming and Organization Rules

1. **File Naming Standards:**
   - Use consistent Title-Case filenames separated by hyphens (not spaces or underscores).
   - Format: `[Vulnerability-Name]-[Testing/Review/Checklist].md`
   - Examples: `IDOR-Security-Testing.md`, `XSS-Testing-Methods.md`, `Django-Security-Review.md`.
2. **One Topic Per File:**
   - Give each distinct vulnerability class, bypass technique, or technology review its own dedicated Markdown file.
3. **Directory Placement:**
   - Place notes in their corresponding numbered category directory:
     - `01-Reconnaissance/`: Discovery, mapping, Google/GitHub/Shodan dorking.
     - `02-Web-Vulnerabilities/`: Client and server-side vulnerabilities (XSS, IDOR, SSRF, File Upload, RCE, etc.).
     - `03-Authentication-and-Account-Security/`: Login, registration, 2FA, password reset, session handling, ATO.
     - `04-Technology-Specific/`: Framework and platform security reviews (Django, Symfony, Jira, EXIF).
     - `05-Research-and-Field-Notes/`: Practical field wisdom, operational tips, and master checklists.
     - `06-Learning-Resources/`: Roadmaps, training syllabi, and practice trackers.
4. **README Navigation:**
   - Whenever you add a new note, update the root [`README.md`](README.md) table of contents with a relative link and a one-line description.

---

## 📝 Document Structure Template

Each new note should follow the standard structure established across the repository:

```markdown
# 📁 [Vulnerability or Topic Title]

*Contributed by [Your Name or Handle] | Educational Bug Bounty Series*

> **Scope and ethics:** [A clear reminder of authorized testing, synthetic test accounts, and scope compliance].

---

## 📌 Executive Summary
[Brief description of the vulnerability mechanism, why it occurs, and business/technical impact].

## 🔍 Core Testing Methodology
[Step-by-step methodology explaining how to identify and test for the issue systematically].
1. Step 1...
2. Step 2...

## 💡 High-Value Bug Bounty Tips & Edge Cases
[Practical field tips, subtle bypasses, parameters to watch, headers, and nuance].
- Tip 1...
- Tip 2...

## 🛠️ Educational Checklists & Safe Examples
[Redacted HTTP requests/responses, checklist checkboxes, or test payloads using safe placeholders].

- [ ] Item 1
- [ ] Item 2

## 🛡️ Mitigation & Remediation
[Actionable defense recommendations for developers and engineering teams].

## 📚 Source Notes & References
[Attribution to original research, RFCs, OWASP references, or source materials].
```

---

## 🔒 Educational Safety & Payload Policy

- **No Malicious Payloads:** Do not submit live destructive exploit payloads, weaponized shellcode, or malicious automated scripts.
- **Neutral Placeholders:** Always redact sensitive values and use safe placeholder domains:
  - URLs: `https://example.com`, `https://test-target.local`
  - Headers / Tokens: `Authorization: Bearer [REDACTED_TEST_TOKEN]`
  - User identifiers: `[Test-Account-A]`, `[Test-Account-B]`
  - IP Addresses: RFC 5737 documentation blocks (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) or `127.0.0.1`.

---

## 📖 Source Attribution & Intellectual Property

- Always give credit to original security researchers, CVE discoverers, and community references.
- Distinguish between your original explanations and summaries derived from external resources.
- Do not paste copyrighted or proprietary content without permission.

---

## 🚀 Contribution Workflow

1. **Fork the repository** on GitHub.
2. **Create a topic branch:**
   ```bash
   git checkout -b feature/add-ssrf-testing-guide
   ```
3. **Write and format your note** following the template and guidelines above.
4. **Update `README.md`** to link your new file in the appropriate category table.
5. **Commit your changes:**
   ```bash
   git commit -m "docs: add SSRF security testing methodology guide"
   ```
6. **Push and open a Pull Request:**
   - Provide a clear summary of your contribution in the PR description.
   - Confirm that all payloads are safe and placeholders are redacted.
