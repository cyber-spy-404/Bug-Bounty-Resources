# 📁 Arbitrary File Upload — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Unrestricted file uploads can lead to full Remote Code Execution (RCE) or persistent Cross-Site Scripting (XSS). Test only systems within your authorized scope. **Do not upload malicious web shells, destructive scripts, or reverse shells.** Use safe, non-destructive proofs-of-concept (such as echoing benign strings or harmless `phpinfo()` calls) and immediately delete uploaded test files after verification.

---

## 📌 Executive Summary

**Arbitrary File Upload vulnerabilities** occur when a web application accepts user-supplied files without adequately verifying their file extension, MIME type, file contents, or storage location. If the uploaded file is stored in a web-accessible directory and the web server is configured to execute scripts (such as PHP, ASPX, JSP, or SSI), an attacker can achieve **Remote Code Execution (RCE)**.

Even if server-side code execution is blocked, malicious uploads can trigger:
- **Stored Cross-Site Scripting (XSS):** Uploading `.svg` or `.html` files containing JavaScript.
- **Server-Side Request Forgery (SSRF) / XXE:** Exploiting backend document processors (PDF converters, Office XML parsers, ImageMagick).
- **Filesystem Overwrite / Path Traversal:** Overwriting critical configuration files through crafted filenames like `../../app.conf`.

---

## 🔍 Core Testing Workflow

```
+---------------------------------------------------------------------------------+
|                       File Upload Verification Flow                             |
+---------------------------------------------------------------------------------+
       |
       v
1. Baseline Test: Upload benign valid image (avatar.png)
       |
       v
2. Storage Recon: Locate uploaded file URL (e.g. /uploads/avatar.png vs S3 CDN)
       |
       v
3. Client-Side Bypass: Intercept request in proxy, alter extension / MIME
       |
       v
4. Validation Probing: Test blacklist, whitelist, magic bytes, and polyglots
       |
       v
5. Execution Verification: Request uploaded file URL to confirm execution vs plaintext
```

---

## 💡 High-Value Bug Bounty Tips & Filter Bypasses

### 1. Extension Blacklist Bypasses
If the application blocks `.php` or `.jsp`, check if alternative executable extensions are permitted:

| Environment | Primary Target | Alternative Executable Extensions |
| :--- | :--- | :--- |
| **PHP** | `.php` | `.phtml`, `.php3`, `.php4`, `.php5`, `.php7`, `.pht`, `.phar`, `.pgif`, `.inc` |
| **ASP.NET** | `.aspx` | `.asp`, `.ashx`, `.asmx`, `.axd`, `.cer`, `.asa`, `.asax` |
| **Java** | `.jsp` | `.jspx`, `.jsw`, `.jsv`, `.jspf`, `.war` |
| **Perl / CGI** | `.cgi` | `.pl`, `.pm`, `.al` |
| **Server-Side Includes**| `.shtml` | `.stm`, `.shtm` |

### 2. Case Sensitivity & Special Characters
Web servers and operating systems handle case sensitivity differently (Windows is case-insensitive; Linux is case-sensitive):
- `shell.PhP`, `shell.pHP`, `shell.AsPx`
- Trailing dot/space (Windows): `shell.php.`, `shell.php `
- URL-encoded characters: `shell.php%20`, `shell.php%0a`
- Semicolon trick (IIS): `shell.asp;.jpg`, `shell.asp;image.png`

### 3. Double Extensions & Reverse Proxy Mismatches
When Apache or Nginx is misconfigured to evaluate multiple extensions:
- `shell.php.jpg` (Apache may execute `.php` if `.jpg` handler is missing)
- `shell.png.php` (Frontend checks `.png`, backend executes `.php`)

### 4. MIME-Type & Content-Type Manipulation
Applications frequently check only the client-supplied `Content-Type` header without examining file contents:
```http
POST /api/upload HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="test_safe.php"
Content-Type: image/jpeg

<?php echo "Safe Test Verification"; ?>
------WebKitFormBoundary--
```

### 5. Magic Bytes & File Header Inspection
When backend code inspects file headers using utilities like `file` or PHP `mime_content_type`:
- Prepend valid image magic bytes to your text file:
  - **GIF89a Header:** `GIF89a;<?php phpinfo(); ?>`
  - **PNG Header:** `\x89PNG\r\n\x1a\n` followed by payload
  - **JPEG Header:** `\xFF\xD8\xFF\xE0` followed by payload

### 6. Stored XSS via Scalable Vector Graphics (SVG)
When testing image upload functions that accept SVG images:
```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">
  <polygon id="triangle" points="0,0 0,50 50,0" fill="#009900" stroke="#004400"/>
  <script type="text/javascript">
    alert(document.domain);
  </script>
</svg>
```
*Note:* If the SVG is served with `Content-Type: image/svg+xml` without a `Content-Disposition: attachment` header, browsing directly to the file URL executes stored XSS in the victim's session context.

### 7. Filename Path Traversal
If the backend directly uses the `filename` parameter when writing to disk:
```http
Content-Disposition: form-data; name="avatar"; filename="../../public/test.txt"
```
Test with single directories (`../test.txt`) and URL-encoded traversal (`..%2ftest.txt`).

---

## 🛠️ Safe Proof-of-Concept & Verification Matrix

### Safe PHP Non-Destructive Payload
```php
<?php
// Educational Verification Only
echo "--- VULNERABILITY CONFIRMED ---";
echo "Host: " . $_SERVER['HTTP_HOST'];
echo "PHP Version: " . phpversion();
?>
```

### Safe ASPX Non-Destructive Payload
```aspx
<%@ Page Language="C#" %>
<html>
<body>
  <h1>Upload Verification</h1>
  <p>Server Time: <%= DateTime.Now.ToString() %></p>
</body>
</html>
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Strict Whitelist Validation
Only accept approved file extensions and MIME types. Reject anything not explicitly permitted:
```python
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif', 'webp'}

def is_allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS
```

### 2. Rename Files Server-Side to Random UUIDs
Never preserve user-supplied filenames on disk. Always generate a random UUID:
```python
import uuid
import os

def generate_safe_filename(original_extension):
    safe_name = f"{uuid.uuid4().hex}.{original_extension.lower()}"
    return safe_name
```

### 3. Store Uploads Outside the Web Root or on Object Storage
- Store user files in cloud object storage (Amazon S3, Google Cloud Storage, Azure Blob Storage) configured as private assets.
- If storing locally, store outside the server's public document root (e.g., `/var/data/uploads/` instead of `/var/www/html/uploads/`).
- Serve files through a controlled download proxy script that sets security headers:
  ```http
  Content-Disposition: attachment; filename="document.pdf"
  X-Content-Type-Options: nosniff
  Content-Security-Policy: default-src 'none'
  ```

### 4. Disable Execution in Upload Directories (Web Server Config)
For Apache (`.htaccess` in upload directory):
```apache
# Prevent script execution in upload directory
php_flag engine off
Options -ExecCGI
AddHandler cgi-script .php .phtml .pl .py .jsp .asp .htm .shtml .sh .cgi
```
For Nginx:
```nginx
location /uploads/ {
    location ~ \.(php|phtml|phar|pl|cgi|asp|aspx|jsp)$ {
        deny all;
        return 404;
    }
}
```

---

## 📊 CVSS 3.1 Severity Reference

- **Remote Code Execution via Web Shell:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H` (Score: **9.9 - Critical**)
- **Stored Cross-Site Scripting via SVG:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N` (Score: **5.4 - Medium**)
- **Arbitrary File Overwrite via Path Traversal:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:H` (Score: **8.1 - High**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Validated that the upload endpoint is within written scope.
- [ ] Uploaded exclusively safe, non-destructive test scripts.
- [ ] Confirmed execution by navigating to the file URL and verifying custom benign output.
- [ ] Tested extension variations, MIME types, and magic bytes.
- [ ] Deleted all test files immediately after confirming the finding.
- [ ] Included exact HTTP request, response, and storage URL in disclosure report.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice uploading payloads, bypassing MIME checks, and achieving RCE safely:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Remote code execution via web shell upload](https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload) | Apprentice | Uploading basic `.php` shell without file type restrictions |
| **PortSwigger Academy** | [Web shell upload via Content-Type restriction bypass](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-content-type-restriction-bypass) | Practitioner | Tampering with `Content-Type: image/jpeg` in Burp Suite |
| **PortSwigger Academy** | [Web shell upload via path traversal](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-path-traversal) | Practitioner | Writing files outside non-executable directories (`..%2f`) |
| **PortSwigger Academy** | [Web shell upload via extension blacklist bypass](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-extension-blacklist-bypass) | Practitioner | Overriding Apache `.htaccess` to execute custom extension files |
| **PortSwigger Academy** | [Web shell upload via obfuscated file extension](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-obfuscated-file-extension) | Practitioner | Null byte injection (`shell.php%00.jpg`) and double extensions |
| **TryHackMe** | [Upload Vulnerabilities Room](https://tryhackme.com/room/uploadvulnerabilities) | Medium | Magic byte editing, client-side JS bypass, and polyglots |
| **Hack The Box** | [HTB Machine: Pilgrimage](https://app.hackthebox.com/machines/Pilgrimage) | Easy | ImageMagick arbitrary file read via crafted PNG profile |
