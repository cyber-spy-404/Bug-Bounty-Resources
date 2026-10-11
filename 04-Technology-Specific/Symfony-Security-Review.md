# 📁 Symfony Framework — Security Review & Audit Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Auditing Symfony applications involves evaluating developer profilers, template engines, and firewall configurations. Test strictly on authorized targets. **Never use disclosed profiler tokens or database credentials to extract customer records or pivot into backend systems.** Report confirmed profiler exposures responsibly.

---

## 📌 Executive Summary

**Symfony** is a widely adopted PHP web framework and component ecosystem (powering major platforms like Drupal, Magento, PrestaShop, and Laravel).

While Symfony provides robust built-in security features through its Security Bundle and Twig template engine, deployment errors—particularly exposing the **Symfony Web Profiler (`_profiler`)** or leaking the **`APP_SECRET`**—expose critical vulnerabilities that can lead directly to full database disclosure or Remote Code Execution (RCE).

```
+---------------------------------------------------------------------------------+
|                        Symfony Security Review Vectors                          |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  Web Profiler Exposure            APP_SECRET Exposure             Twig SSTI Injections
  - /_profiler/open?file=          - Fragment URL signing          - {{7*7}} -> 49
  - Full DB queries & env vars     - Code execution via _fragment   - Filter callbacks
```

---

## 🔍 Core Symfony Vulnerability Vectors

### 1. Exposed Symfony Web Profiler (`/_profiler`)
The Symfony Web Profiler is an in-depth debugging tool intended exclusively for local development. If deployed with `APP_ENV=dev` or if route access is left unauthenticated in production:

#### Critical Profiler Endpoints to Probe:
- `/_profiler/` (Main profiler dashboard and search)
- `/_profiler/empty/search/results?limit=10` (Recent request tokens)
- `/_wdt/{token}` (Web Debug Toolbar HTML fragment)
- `/_profiler/open?file=app/config/parameters.yml` (Arbitrary file reader on misconfigured versions)

#### Information Disclosed by the Profiler:
- **Database Queries:** Every raw SQL query executed during the request, including hashed or plaintext passwords.
- **Environment & Configuration:** Full output of `$_ENV` and `$_SERVER` (Database passwords, AWS keys, mailer DSNs).
- **Session Tokens:** Active session identifiers and security tokens for all recent visitors.
- **Routing Table:** Complete internal list of hidden admin and debugging endpoints.

---

### 2. `APP_SECRET` Exposure & Fragment RCE
In Symfony, `APP_SECRET` is used to cryptographically sign fragment URLs (used for ESI / internal sub-requests via `/_fragment`):
- If `APP_SECRET` is leaked (via `.env`, profiler, or GitHub dorking), an attacker can sign arbitrary fragment URLs:
  ```http
  GET /_fragment?_hash=COMPUTED_HMAC_HASH&_path=_controller%3Dsystem%26command%3Did HTTP/1.1
  Host: example.com
  ```
- *Impact:* Leads to **Remote Code Execution (RCE)** by invoking arbitrary PHP callable functions through the fragment controller.

---

### 3. Twig Server-Side Template Injection (SSTI)
Symfony uses the Twig template engine. If user input is concatenated into template strings:
- **Detection Payload:** `{{7*7}}` or `{{7*'7'}}`
  - If the page renders `49`, Twig SSTI is confirmed.
- **Information Gathering Payload:** `{{_self.env}}`
- **Mitigation:** Never pass dynamic user strings to `$twig->createTemplate($userInput)`. Always render static template files passing user input as context parameters:
  ```php
  // ✅ SECURE
  return $this->render('profile.html.twig', ['name' => $userName]);
  ```

---

### 4. Firewall & `security.yaml` Path Ordering Flaws
Symfony's security firewall evaluates access control rules from **top to bottom**, stopping at the first rule that matches:
```yaml
# ❌ VULNERABLE: Overly broad rule placed before specific rule
access_control:
    - { path: ^/, roles: PUBLIC_ACCESS }            # Matches EVERYTHING first!
    - { path: ^/admin, roles: ROLE_ADMIN }          # NEVER REACHED!
```
```yaml
# ✅ SECURE: Specific rules placed FIRST
access_control:
    - { path: ^/admin, roles: ROLE_ADMIN }
    - { path: ^/api/v1/private, roles: ROLE_USER }
    - { path: ^/, roles: PUBLIC_ACCESS }
```

---

## 🛠️ Safe Educational Verification Template

### Safe HTTP Request (Probing for Profiler Dashboard)
```http
GET /_profiler/ HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
```

### Vulnerable Response Body
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="en">
<head>
  <title>Symfony Profiler</title>
</head>
<body>
  <h1>Profiler Dashboard</h1>
  <p>Symfony Profiler token search...</p>
</body>
</html>
```

---

## 🛡️ Production Hardening Checklist for Symfony

### 1. Set Production Environment in `.env.local`
Ensure production servers operate strictly under the `prod` environment with debugging disabled:
```ini
# .env.local
APP_ENV=prod
APP_DEBUG=0
```

### 2. Remove Debug Bundles from Production
In `config/bundles.php`, ensure debug packages are registered **only** for `dev` and `test` environments:
```php
return [
    Symfony\Bundle\FrameworkBundle\FrameworkBundle::class => ['all' => true],
    Symfony\Bundle\WebProfilerBundle\WebProfilerBundle::class => ['dev' => true, 'test' => true],
    Symfony\Bundle\DebugBundle\DebugBundle::class => ['dev' => true, 'test' => true],
];
```

### 3. Web Server Access Denial (Nginx)
Explicitly block access to profiler routes at the web server reverse proxy level:
```nginx
location ~ ^/(_profiler|_wdt|_error) {
    deny all;
    return 404;
}
```

---

## 📊 CVSS 3.1 Severity Reference

- **Publicly Exposed Symfony Web Profiler (`/_profiler`):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (Score: **7.5 - High**)
- **`APP_SECRET` Exposure leading to Fragment RCE:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (Score: **10.0 - Critical**)
- **Twig Server-Side Template Injection (SSTI):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (Score: **10.0 - Critical**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target endpoint is verified within program scope.
- [ ] Confirmed whether `/_profiler/` or `/_wdt/` returns active debugging dashboards.
- [ ] Checked for Twig SSTI using non-destructive math expression `{{7*7}}`.
- [ ] Stopped immediately upon verifying profiler exposure without harvesting credentials.
- [ ] Redacted all database queries, environment variables, and passwords in report.
