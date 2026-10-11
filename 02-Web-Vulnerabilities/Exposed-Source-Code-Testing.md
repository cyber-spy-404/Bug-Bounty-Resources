# 📁 Exposed Source Code & Sensitive Metadata — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Publicly accessible source code and configuration files often contain sensitive secrets, database connection strings, and architecture details. Test only authorized assets. **Never download, clone, or exfiltrate private codebases or customer data.** Stop as soon as you confirm that metadata is accessible, collect minimal redacted HTTP response evidence, and report immediately.

---

## 📌 Executive Summary

**Exposed Source Code and Metadata vulnerabilities** occur when web servers, reverse proxies, or cloud object storages mistakenly serve private version control directories (such as `.git`), environment variables (`.env`), build source maps (`.js.map`), or editor backup files (`.bak`, `.swp`) to unauthenticated public visitors.

### ⚠️ Impact and Threat Profile
1. **Hardcoded Secrets:** Exposure of production database passwords, cloud API keys (AWS, GCP, Azure), encryption salts, and private keys.
2. **Architecture Exposure:** Attackers gain a comprehensive blueprint of backend routing, unauthenticated internal endpoints, and hidden parameters.
3. **Intellectual Property Leakage:** Proprietary algorithms, business logic, and custom security mechanisms are fully disclosed.

---

## 🔍 Core Discovery & Testing Methodology

```
+---------------------------------------------------------------------------------+
|                       Exposed Metadata Discovery Vectors                        |
+---------------------------------------------------------------------------------+
          |                                  |                              |
          v                                  v                              v
  Version Control                    Environment & Backups            Frontend Artifacts
  - /.git/HEAD                       - /.env                          - app.bundle.js.map
  - /.git/config                     - /config.json.bak               - webpack:// sources
  - /.svn/entries                    - /web.config.old                - Swagger JSON dumps
```

---

## 🧭 High-Priority Sensitive Endpoints & File Signatures

### 1. Version Control Metadata
Developers copying files directly to production servers or failing to configure web server block rules often leave version control directories exposed:

| Endpoint Path | File Contents / Signature | Verification Technique |
| :--- | :--- | :--- |
| `/.git/HEAD` | `ref: refs/heads/main` or `ref: refs/heads/master` | Check status `200 OK` and plaintext `ref:` header |
| `/.git/config` | `[core] repositoryformatversion = 0 ...` | Contains remote repository URLs and branch info |
| `/.svn/entries` | Subversion XML/Text entries metadata | Displays revision numbers and repository URLs |
| `/.hg/dirstate` | Mercurial repository manifest | Binary/plaintext Mercurial signatures |
| `/.bzr/README` | Bazaar version control metadata | Identifies Bazaar branch metadata |

### 2. Environment & Configuration Files
Framework configuration files containing environment variables:

| Endpoint Path | Expected Signature | Severity |
| :--- | :--- | :--- |
| `/.env`, `/.env.local`, `/.env.prod` | `DB_PASSWORD=`, `SECRET_KEY=`, `APP_ENV=` | **Critical** |
| `/config.json`, `/config.yml` | Application database credentials, API endpoints | **High** |
| `/web.config`, `/settings.py` | Microsoft IIS / Django configuration blocks | **High** |
| `/docker-compose.yml`, `/Dockerfile` | Container architecture, service links, internal ports | **Medium** |

### 3. Editor & Temporary Backup Files
Text editors and text backup operations often generate temporary files that web servers fail to interpret as executable scripts, instead serving them as plaintext:

| File Pattern | Origin / Tool | Risk |
| :--- | :--- | :--- |
| `/index.php.bak`, `/index.php.old` | Manual developer backup before patching | Discloses raw PHP source code |
| `/index.php~`, `/.index.php.swp` | Vim / Nano editor crash recovery files | Exposes raw unrendered code |
| `/backup.zip`, `/site.tar.gz` | Forgotten site archive left in web root | Complete filesystem exposure |

### 4. JavaScript Source Maps (`.js.map`)
Modern Single Page Applications (React, Vue, Angular) are bundled using Webpack, Vite, or Rollup. If production source maps are not disabled:
1. Identify bundled files in DevTools: `https://example.com/static/js/main.a48d91b.js`.
2. Append `.map`: `https://example.com/static/js/main.a48d91b.js.map`.
3. If accessible, the entire original uncompiled source tree (including developer comments, internal API routes, and hidden features) can be reconstructed.

---

## 🛠️ Safe Testing & Proof-of-Concept Workflow

### Step 1: Send Minimal Verification Request
Use `HEAD` or a byte-range `GET` request to minimize server impact and avoid downloading file contents:
```http
HEAD /.git/HEAD HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```

### Step 2: Confirm Response Header & Content Type
```http
HTTP/1.1 200 OK
Date: Sun, 11 Oct 2026 12:00:00 GMT
Content-Type: text/plain
Content-Length: 23

ref: refs/heads/main
```

### Step 3: Stop and Document Minimal Evidence
- Record the URL, HTTP Status, Content-Type, and the first few lines of the file.
- **DO NOT** use automated Git dumpers (`git-dumper`, `rip-git`) on live bug bounty targets without explicit program authorization. Running extractors can trigger severe security alarms, degrade availability, and violate privacy boundaries.

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Nginx Web Server Hardening
Block access to all hidden files, `.git`, `.env`, and backup files:
```nginx
# Block all hidden files (.git, .env, .htaccess, etc.)
location ~ /\. {
    deny all;
    access_log off;
    log_not_found off;
    return 404;
}

# Block backup, temp, and config extensions
location ~* \.(bak|config|sql|fla|psd|ini|log|sh|inc|swp|dist|old)$ {
    deny all;
    return 404;
}
```

### 2. Apache HTTP Server Hardening
Add to `/etc/apache2/apache2.conf` or `.htaccess`:
```apache
# Deny access to .git and hidden directories
<DirectoryMatch "/\.(git|svn|hg)">
    Require all denied
</DirectoryMatch>

# Block environment and backup files
<FilesMatch "(\.env|\.bak|\.old|\.swp|\.config)$">
    Require all denied
</FilesMatch>
```

### 3. CI/CD Deployment Hygiene
- Never deploy the working `.git` directory to production web roots.
- Use multi-stage Docker builds or deployment pipelines that package only compiled build artifacts.
- Set `GENERATE_SOURCEMAP=false` (in React/Vue production builds) to prevent uploading `.js.map` files to public CDNs.

---

## 📊 CVSS 3.1 Severity Reference

- **Exposed `.env` with Live Cloud / Database Credentials:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (Score: **10.0 - Critical**)
- **Exposed `.git` Directory (Full Source Disclosure):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (Score: **7.5 - High**)
- **Exposed Frontend Source Maps (`.js.map`):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` (Score: **5.3 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target host and path are verified in scope.
- [ ] Confirmed status is `200 OK` (not a customized 404 page returning 200).
- [ ] Checked that response body contains valid metadata signatures (`ref: refs/heads/`, `DB_PASSWORD=`).
- [ ] Stopped immediately without downloading full repositories or databases.
- [ ] Redacted all confirmed secrets or proprietary paths in the disclosure report.
