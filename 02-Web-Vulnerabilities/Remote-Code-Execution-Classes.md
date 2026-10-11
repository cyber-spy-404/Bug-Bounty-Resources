# 📁 Remote Code Execution (RCE) — Vulnerability Classes & Methodology Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Remote Code Execution is the most critical vulnerability category in application security. Testing must remain strictly within written program scope. **Never drop persistent backdoors, deploy automated botnets, exfiltrate private credentials, or execute destructive operating system commands.** Use safe, non-destructive proofs-of-concept (such as non-intrusive mathematical evaluation, controlled DNS callbacks, or benign sleep delays).

---

## 📌 Executive Summary

**Remote Code Execution (RCE)** occurs when an attacker can execute arbitrary instructions, shell commands, or backend code on the server hosting an application. While RCE represents the final impact, the root causes span diverse vulnerability classes across different programming languages and architecture layers.

Understanding the underlying vulnerability class is essential for designing safe verification payloads and providing accurate remediation advice.

```
+---------------------------------------------------------------------------------+
|                       Remote Code Execution (RCE) Classes                       |
+---------------------------------------------------------------------------------+
        |                        |                         |                |
        v                        v                         v                v
  OS Command Inj.         Template Inj. (SSTI)       Deserialization    Code Eval / LFI
  - system(), exec()      - Jinja2, Twig, Velocity   - Java readObject  - eval(), $var()
  - Shell metacharacters  - Math: {{7*7}} -> 49      - Python pickle    - php://input / wrappers
```

---

## 🔍 The 5 Core Vulnerability Classes Leading to RCE

### 1. Operating System Command Injection
Occurs when an application constructs operating system commands by concatenating unsanitized user input into system shells (`/bin/sh`, `cmd.exe`).
- **Dangerous Functions:** `system()`, `exec()`, `shell_exec()`, `popen()`, `proc_open()` (PHP/C); `os.system()`, `subprocess.Popen(..., shell=True)` (Python); `Runtime.getRuntime().exec()` (Java).
- **Injection Mechanisms:** Command separators (`;`, `&`, `&&`, `|`, `||`, newline `\n`, backticks `` ` ``).
- *See dedicated guide:* [`Command-Injection-Security-Testing.md`](Command-Injection-Security-Testing.md).

---

### 2. Server-Side Template Injection (SSTI)
Occurs when user input is concatenated directly into template engine strings rather than passed as data parameters to a compiled template.
- **Affected Engines:** Jinja2 (Python), Twig (PHP), FreeMarker (Java), Thymeleaf (Java), Velocity (Java), Smarty (PHP), ERB (Ruby).

#### Safe Detection Workflow:
Inject non-destructive mathematical expressions and observe if the output evaluates the result:

| Test Payload | Interpreted By | Expected Reflection |
| :--- | :--- | :--- |
| `{{7*7}}` | Jinja2, Twig, Vue, AngularJS | `49` |
| `${7*7}` | FreeMarker, Thymeleaf, Spring EL | `49` |
| `<%= 7*7 %>` | ERB (Ruby) | `49` |
| `#{7*7}` | Java Expression Language (EL) | `49` |

```
               Inject: "{{7*7}}"
                      |
                      v
        +----------------------------+
        | Does template reflect "49"?|
        +-------------+--------------+
                      |
           +----------+----------+
           | Yes                 | No
           v                     v
    Test "{{7*'7'}}"        Test "${7*7}"
      |          |
      |          +-> "7777777" (Jinja2)
      +-> "49" (Twig)
```

---

### 3. Insecure Deserialization
Occurs when an application reconstructs serialized objects from untrusted input without validating the object graph, enabling gadget chain execution.
- **Java:** `ObjectInputStream.readObject()` with libraries containing gadget chains (Apache Commons Collections, Spring, Jackson).
- **Python:** `pickle.loads()`, `_pickle`, or `yaml.load(..., Loader=yaml.Loader)`.
- **PHP:** `unserialize()` abusing magic methods (`__wakeup`, `__destruct`, `__toString`).
- **Ruby:** `Marshal.load()` or `YAML.load()`.
- **Safe Detection:** Look for base64-encoded serialized signatures:
  - Java Serialization Magic Bytes: `\xac\xed\x00\x05` (Base64: `rO0AB...`).
  - Python Pickle: `\x80\x03` or `\x80\x04` (Base64: `gASV...`).
  - PHP Serialized Objects: `O:4:"User":...` or `a:2:{...}`.

---

### 4. File Inclusion & Dynamic Execution (LFI / RFI)
Occurs when user input controls file paths loaded into the execution context:
- **PHP:** `include()`, `require()`, `include_once()`.
  - PHP Wrappers: `php://filter/convert.base64-encode/resource=index.php` (Discloses source).
  - Data Wrappers: `data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8+` (If `allow_url_include=On`).
  - Apache / Nginx Log Poisoning: Injecting PHP tags into the `User-Agent` and including the access log.

---

### 5. Unsafe Code Evaluation & Dynamic Reflection
Occurs when user input is supplied to dynamic language interpreters:
- **PHP:** `eval()`, `assert()`, preg_replace with `/e` modifier.
- **JavaScript / Node.js:** `eval()`, `new Function(userInput)()`, `setTimeout(userInput)`.
- **Python:** `eval()`, `exec()`, `compile()`.

---

## 🛠️ Safe Proof-of-Concept & Verification Rules

To maintain ethical standards and prevent service outages:

### 1. The "Math First" Rule
In SSTI and Code Eval, always verify using arithmetic (`7*7` -> `49` or `1337*2` -> `2674`). Never jump directly to shell commands.

### 2. Time-Based Non-Destructive Delay
When testing blind RCE where no output is returned:
- Inject a controlled delay (e.g., `sleep 5`).
- Compare response duration:
  - Baseline request: `120 ms`
  - Injected request: `5,140 ms` -> High probability of blind execution.
- Test once with 5 seconds and once with 10 seconds to confirm consistency.

### 3. Out-of-Band (OOB) DNS Pingback
Use non-intrusive DNS lookup utilities (Burp Collaborator, Interactsh, or a private DNS server):
```bash
# Safe OOB probe command
nslookup [CANARY_SUBDOMAIN].your-domain.com
curl http://[CANARY_SUBDOMAIN].your-domain.com
```

---

## 🛡️ Defense-in-Depth Remediation Architecture

### 1. Eliminate Shell Invocations
Never use system shells. Use parameterized API execution where arguments cannot be interpreted as command separators:
```python
# ❌ VULNERABLE: Invokes shell
os.system(f"convert {user_filename} output.png")

# ✅ SECURE: Passes arguments as a strict array without shell
import subprocess
subprocess.run(["/usr/bin/convert", user_filename, "output.png"], check=True)
```

### 2. Treat Templates as Logic-Less Views
- Use logic-less or sandboxed template engines.
- Never concatenate user strings into template sources:
```python
# ❌ VULNERABLE
template = Template(f"Hello {user_name}")

# ✅ SECURE: Pass user_name as template context variable
template = env.get_template("greeting.html")
rendered = template.render(user_name=user_name)
```

### 3. Safe Serialization Formats
- Replace native language serialization (`pickle`, Java serialization, PHP serialize) with language-agnostic data serialization formats (**JSON**, **Protocol Buffers**, or **MessagePack**).
- If native deserialization is unavoidable, implement cryptographic message signing (HMAC-SHA256) or strict class filtering (Java `ValidatingObjectInputStream`).

---

## 📊 CVSS 3.1 Severity Reference

- **Unauthenticated Remote Code Execution:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (Score: **10.0 - Critical**)
- **Authenticated RCE (Low Privilege):** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H` (Score: **9.9 - Critical**)
- **Local File Inclusion (Sensitive File Read):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (Score: **7.5 - High**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Identified exact vulnerability class (OS Command, SSTI, Deserialization, LFI).
- [ ] Used strictly non-destructive proofs-of-concept (arithmetic, sleep delay, DNS canary).
- [ ] Documented clear step-by-step reproduction instructions.
- [ ] Stopped immediately upon confirming command execution without reading system files or pivoting.
- [ ] Recommended structural architecture remediation (parameterization, safe deserialization).
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice Server-Side Template Injection (SSTI), Deserialization, and RCE classes:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [Basic server-side template injection](https://portswigger.net/web-security/server-side-template-injection/lab-server-side-template-injection-basic) | Apprentice | Identifying ERB template evaluation `${7*7}` in message templates |
| **PortSwigger Academy** | [Server-side template injection using documentation](https://portswigger.net/web-security/server-side-template-injection/lab-server-side-template-injection-using-documentation) | Practitioner | Exploiting FreeMarker template syntax to execute system commands |
| **PortSwigger Academy** | [Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects) | Apprentice | PHP serialized cookie manipulation to achieve administrative access |
| **TryHackMe** | [Server-Side Template Injection (SSTI)](https://tryhackme.com/room/server-side-template-injection) | Medium | Jinja2, Twig, and Smarty sandbox escapes leading to RCE |
| **Hack The Box** | [HTB Machine: Sau](https://app.hackthebox.com/machines/Sau) | Easy | Request-baskets SSRF chained to Malta RCE |
