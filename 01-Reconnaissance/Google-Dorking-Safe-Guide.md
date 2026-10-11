# 📁 Google Dorking — Advanced Search Intelligence & Safe Recon Guide

*Contributed by Cyber_Spy.404 | Bug Bounty Notes*

> **Scope and ethics:** Google Dorking leverages publicly indexed search engine records to discover exposed files and forgotten portals. Target only organizations and domains covered by your authorized bug bounty scope. **Never download, exfiltrate, or publish sensitive third-party spreadsheets, private keys, or customer documents discovered through search engines.** Disclose the exposed URL or index listing responsibly to the program team.

---

## 📌 Executive Summary

**Google Dorking** (also known as *Google Hacking*) utilizes advanced search engine filter operators to uncover sensitive data, misconfigured servers, forgotten staging environments, and administrative portals that search engine crawlers (Googlebot) have indexed from the public web.

Because web servers often serve directories without authentication or lack appropriate `robots.txt` and `noindex` directives, crawlers index internal assets, making them discoverable to anyone with the right query syntax.

```
+---------------------------------------------------------------------------------+
|                       Google Advanced Operator Syntax                           |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
    site:target.com                 filetype:sql / ext:env          intitle:"index of"
  - Scopes query to domain         - Filters by file extension      - Finds open directories
```

---

## 🔍 Google Advanced Search Operators Dictionary

| Operator | Purpose | Practical Bug Bounty Example |
| :--- | :--- | :--- |
| **`site:`** | Restricts results strictly to the designated domain or wildcard subdomain. | `site:example.com` or `site:*.example.com` |
| **`inurl:`** | Finds pages containing the keyword inside the URL path or query string. | `site:example.com inurl:admin` |
| **`allinurl:`** | Requires all specified terms to appear in the URL path. | `site:example.com allinurl:auth login` |
| **`intitle:`** | Searches for terms in the HTML `<title>` tag. | `site:example.com intitle:"Index of /"` |
| **`allintitle:`**| Requires all terms to appear in the `<title>` tag. | `site:example.com allintitle:Dashboard Staging` |
| **`intext:`** | Searches for keywords within the visible body text. | `site:example.com intext:"API_KEY"` |
| **`allintext:`** | Requires all keywords to appear in the page text. | `site:example.com allintext:username password filetype:log` |
| **`filetype:`** | Restricts search to a specific file extension (interchangeable with `ext:`). | `site:example.com filetype:pdf` |
| **`-` (Negation)** | Excludes specific terms, subdomains, or file extensions from results. | `site:example.com -site:www.example.com` |
| **`""` (Quotes)** | Enforces exact phrase matching without synonym expansion. | `site:example.com "-----BEGIN RSA PRIVATE KEY-----"` |
| **`OR` / `\|`** | Logical OR operator (must be capitalized). | `site:example.com ext:env OR ext:yml` |

---

## 💡 High-Value Bug Bounty Dorking Categories

### 1. Subdomain Discovery (Eliminating Known Roots)
Find forgotten or obscure subdomains indexed by Google by systematically negating discovered assets:
```text
site:example.com -site:www.example.com -site:blog.example.com -site:shop.example.com
```

---

### 2. Open Directory Indexing ("Index of /")
Web servers with directory browsing enabled frequently expose certificate files, source archives, and keys:
```text
site:example.com intitle:"index of"
site:example.com intitle:"index of /" "*key.pem"
site:example.com intitle:"index of" "config.db"
site:example.com intitle:"index of" "backup"
```

---

### 3. Exposed Environment & Configuration Files
Discover `.env`, YAML, and INI configuration files that developers accidentally uploaded to public web roots:
```text
site:example.com ext:env OR ext:yml OR ext:ini
site:example.com "MYSQL_ROOT_PASSWORD:" ext:env OR ext:yml -git
site:example.com allintext:"API_SECRET*" ext:env | ext:yml
site:example.com inurl:wp-config.php.bak
```

---

### 4. Database Exports & SQL Dumps
```text
site:example.com ext:sql OR ext:db OR ext:sqlite OR ext:dump
site:example.com intext:admin ext:sql inurl:admin
site:example.com filetype:sql "INSERT INTO"
```

---

### 5. Log Files Containing Plaintext Credentials
```text
site:example.com ext:log
site:example.com allintext:username,password filetype:log
site:example.com inurl:log "DEBUG"
```

---

### 6. Third-Party Code & Document Sharing Leaks
Search external collaboration platforms for target proprietary tokens or documents:
```text
site:pastebin.com "example.com"
site:trello.com "example.com"
site:docs.google.com "example.com" "confidential"
site:codepad.co "example.com"
site:npmjs.com "example.com"
site:*atlassian.net "example.com"
```

---

## 🛠️ Safe Verification & Reporting Workflow

1. **Verify Indexing Only:** Click the link in Google search results to verify that the URL is accessible without authentication.
2. **Minimal Evidence:** Take a screenshot of the search result and the initial page header.
3. **Do Not Scrape:** Never write scripts to scrape or dump all files found via Google.
4. **Report Responsibly:** Provide the exact Google dork query and the specific exposed URL to the program security team.

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Enforce `X-Robots-Tag: noindex, nofollow`
A common misconception is that `robots.txt` hides pages. In reality, `robots.txt` merely prevents crawling; pages can still be indexed if linked elsewhere!
To completely prevent indexing, configure your web server to send the `X-Robots-Tag` header:
```nginx
# Nginx: Block indexing on sensitive routes
location /internal/ {
    add_header X-Robots-Tag "noindex, nofollow, nosnippet, noarchive";
}
```

### 2. Disable Directory Indexing
- In Apache: `Options -Indexes` in `.htaccess` or virtual host configuration.
- In Nginx: Ensure `autoindex off;` is configured (default behavior).

### 3. Google Search Console URL Removal Tool
If sensitive documents or staging portals have already been indexed by Google:
1. Revoke public access immediately on the origin server (`404` or `401`).
2. Use **Google Search Console -> Removals** to request an expedited temporary removal of the cached URL from Google search results.

---

## 📊 CVSS 3.1 Severity Reference

- **Publicly Indexed `.env` with Live Master Secrets:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (Score: **10.0 - Critical**)
- **Exposed Database Dump (`.sql`) with User Data:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (Score: **7.5 - High**)
- **Subdomain / Admin Portal Discovery via Dorking:** Informational / Low (Used for reconnaissance).

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target domain is confirmed within authorized program scope.
- [ ] Confirmed that the exposed file is publicly reachable without authentication.
- [ ] Captured minimal redacted evidence (URL + status + snippet).
- [ ] Did not exfiltrate or download complete database dumps or confidential documents.
- [ ] Recommended `X-Robots-Tag: noindex` and directory indexing restrictions.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice advanced search operator reconnaissance without risking live IP blocks:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **TryHackMe** | [Google Dorking Room](https://tryhackme.com/room/googledorking) | Easy | Search operator syntax, `site:`, `filetype:`, finding exposed assets |
| **TryHackMe** | [Searchlight - IMINT & OSINT](https://tryhackme.com/room/searchlight) | Medium | OSINT indexing, cache recovery, finding hidden directories |
| **PentesterLab** | [Google Dorks & Information Disclosure](https://pentesterlab.com/) | Easy | Finding backup `.sql`, `.bak`, and configuration files |
| **Hacker101 CTF** | [Photo Gallery Challenge](https://ctf.hacker101.com/) | Medium | Endpoint discovery through public parameter patterns |
