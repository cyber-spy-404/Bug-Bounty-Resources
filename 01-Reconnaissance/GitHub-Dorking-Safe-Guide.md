# 📁 GitHub Dorking — Code Search Reconnaissance & Secret Leak Guide

*Contributed by Cyber_Spy.404 | Bug Bounty Notes*

> **Scope and ethics:** Public GitHub repositories often contain accidentally committed API keys, cloud credentials, and internal configuration files. Target only organizations and domains covered by your authorized bug bounty scope. **Never use discovered credentials to log into production databases, exfiltrate private codebases, or access third-party systems.** Validate credentials only using minimal non-destructive metadata calls (e.g., `aws sts get-caller-identity`) and report the finding immediately.

---

## 📌 Executive Summary

**GitHub Dorking** leverages GitHub's code search engine to discover exposed credentials, API tokens, staging URLs, and internal architecture documentation that developers have inadvertently committed to public repositories.

Because developers frequently work across private corporate organizations and personal public repositories, accidental commits of `.env` files, hardcoded connection strings, and test scripts create critical entry points into corporate infrastructure.

```
+---------------------------------------------------------------------------------+
|                       GitHub Code Search Query Syntax                           |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
   org:target-org                   filename:.env                    "aws_access_key_id"
  - Scopes to company repos        - Targets environment files       - High-entropy secret term
```

---

## 🔍 GitHub Search Operators & Syntax

| Operator | Purpose | Example Syntax |
| :--- | :--- | :--- |
| **`org:`** | Restricts search to repositories owned by a specific GitHub organization. | `org:target-company "password"` |
| **`user:`** | Restricts search to repositories owned by a specific GitHub username. | `user:lead-developer "api_key"` |
| **`repo:`** | Scopes search to a single specific repository. | `repo:target/frontend "token"` |
| **`filename:`** | Searches for specific file names. | `filename:.env "SECRET_KEY"` |
| **`path:`** | Matches files within specific directory paths. | `path:config/ "database"` |
| **`extension:`** | Restricts search to file extensions. | `extension:pem "-----BEGIN"` |
| **`language:`** | Filters by programming language. | `language:python "password"` |
| **`""` (Quotes)** | Matches exact phrases. | `"api.target.com" "Authorization: Bearer"` |

---

## 🔑 High-Entropy Secret Signatures & Queries

### 1. Cloud Provider Credentials (AWS, GCP, Azure)
```text
"target.com" "AKIA" OR "aws_access_key_id"
org:target-org "AWSSecretKey"
"target.com" "api.googlemaps AIza" OR "AIzaSy"
org:target-org "azure_storage_account" OR "AccountKey="
```

### 2. Payment & Third-Party API Keys
```text
"target.com" "sk_live_" (Stripe live secret key)
"target.com" "xoxb-" OR "xoxp-" (Slack bot / user token)
"target.com" "hooks.slack.com/services/" (Slack incoming webhooks)
"target.com" "SG." (SendGrid email API key)
org:target-org "twilio" "auth_token"
```

### 3. Database & Infrastructure Connection Strings
```text
org:target-org "mongodb+srv://" OR "postgres://" OR "mysql://"
"target.com" "db_password" OR "database_password"
"target.com" "jdbc:mysql://"
```

### 4. Configuration & Dotfiles Exposure
```text
org:target-org filename:.env
org:target-org filename:settings.py "SECRET_KEY"
org:target-org filename:wp-config.php "DB_PASSWORD"
"target.com" filename:.bashrc OR filename:.zshrc "export"
"target.com" filename:id_rsa OR filename:id_dsa
```

### 5. Staging & Internal Domain Discovery
```text
"target.com" "staging." OR "internal." OR "dev." OR "test."
"target.com" "vpn." OR "corp."
```

---

## 🛠️ Safe Verification & Triage Workflow

When you discover an exposed credential:

### Step 1: Verify Association with Target Scope
Ensure the repository belongs to the target company, an official contractor, or references in-scope systems.

### Step 2: Minimal Non-Destructive Validation
Confirm whether the credential is active using read-only identity checks:
- **AWS Key:** Run `aws sts get-caller-identity` (returns only the Account ID and ARN; does not read data).
- **GitHub Token:** Run `curl -H "Authorization: token [TOKEN]" https://api.github.com/user` (confirms user identity).
- **Slack Webhook:** Post a harmless test string to your own channel or check URL validity.

### Step 3: Redact and Disclose Immediately
- Record the repository URL, commit hash, and file path.
- In your report, redact the secret (e.g. `AKIAIOSFODNN7E******`).
- Notify the security team so they can revoke and rotate the secret immediately.

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Automated Secret Scanning in CI/CD
Integrate automated secret detection into pre-commit hooks and GitHub Actions:
- **TruffleHog:** Scans git repositories for high-entropy secrets and over 700 token types.
- **Gitleaks:** Lightweight pre-commit secret detection.
```yaml
# GitHub Actions Secret Scanning Workflow (.github/workflows/gitleaks.yml)
name: Gitleaks
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - uses: gitleaks/gitleaks-action@v2
```

### 2. Enable GitHub Secret Scanning & Push Protection
Enable GitHub's native Secret Scanning on all private and public repositories to block pushes containing known token formats automatically.

### 3. Rewriting Git History After Leakage
Simply deleting a file in a new commit does **NOT** remove it from Git history! Anyone can view historical commits.
To completely expunge secrets from history, use **git-filter-repo** or **BFG Repo-Cleaner**:
```bash
git filter-repo --invert-paths --path .env
```
*Crucial Note:* Always assume any leaked secret was harvested by automated scraping bots within seconds of being pushed. **Immediate credential rotation is mandatory.**

---

## 📊 CVSS 3.1 Severity Reference

- **Leaked Production AWS Root / Admin Credentials:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (Score: **10.0 - Critical**)
- **Leaked Stripe Live API Secret (`sk_live_`):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` (Score: **9.1 - Critical**)
- **Leaked Internal Staging / QA Database Passwords:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (Score: **7.5 - High**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target organization or domain is confirmed within authorized scope.
- [ ] Confirmed that repository is public and accessible without authentication.
- [ ] Conducted only minimal non-destructive validation (`sts get-caller-identity`).
- [ ] Captured commit hash, file link, and redacted secret.
- [ ] Advised security team on immediate token revocation and history rewriting.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice secret hunting and source code intelligence safely:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **TryHackMe** | [Git Happens Room](https://tryhackme.com/room/githappens) | Easy | Recovering commit history, finding hardcoded API keys |
| **Hack The Box** | [HTB Machine: Pilgrimage](https://app.hackthebox.com/machines/Pilgrimage) | Easy | Recovering exposed `.git` directories and analyzing source leaks |
| **PentesterLab** | [Git Directory Extraction Badge](https://pentesterlab.com/) | Medium | Reconstructing source repositories from `.git/index` |
| **OWASP** | [crAPI: Secret Discovery Challenge](https://github.com/OWASP/crAPI) | Medium | Identifying leaked third-party keys in public codebases |
