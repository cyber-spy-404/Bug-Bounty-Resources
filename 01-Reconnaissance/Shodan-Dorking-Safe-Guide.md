# 📁 Shodan Search Intelligence — Passive Asset Discovery & Safe Recon Guide

*Contributed by Cyber_Spy.404 | Bug Bounty Notes*

> **Scope and ethics:** Shodan aggregates passive internet-wide scanning data. Searching Shodan is completely passive and does not generate network traffic to the target. However, **connecting to or interacting with discovered ports, unauthenticated databases, or administrative panels requires explicit written scope authorization.** Target only systems covered by your program agreement.

---

## 📌 Executive Summary

**Shodan** is a search engine for internet-connected devices, servers, and services. Unlike Google, which indexes webpage HTML content, Shodan queries banners, SSL certificates, protocol responses, and open network ports (such as HTTP, SSH, FTP, RDP, MySQL, Elasticsearch, and Redis) across the entire IPv4 and IPv6 address space.

For bug bounty hunters, Shodan provides rapid passive reconnaissance to:
- Discover forgotten development, testing, and staging servers.
- Uncover true origin IP addresses hiding behind CDNs or WAFs (Cloudflare bypass).
- Identify unauthenticated databases and internal management portals.

```
+---------------------------------------------------------------------------------+
|                       Shodan Passive Discovery Vectors                          |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
   SSL Certificate Pivoting         Organization & ASN Mapping      Exposed Service Banners
  - ssl:"target.com"               - org:"Target Corp"              - port:9200 "status: 200"
  - Finds true origin IPs          - Discovers owned IP blocks      - Unauthenticated databases
```

---

## 🔍 Shodan Search Filter Operators Dictionary

| Filter | Purpose | Example Syntax |
| :--- | :--- | :--- |
| **`hostname:`** | Matches hostnames and reverse DNS records. | `hostname:"example.com"` |
| **`ssl:`** | Searches across all fields of an SSL certificate. | `ssl:"example.com"` |
| **`ssl.cert.subject.CN:`**| Matches the Common Name (CN) in SSL certificates. | `ssl.cert.subject.CN:"example.com"` |
| **`org:`** | Matches the registered organization name in WHOIS/IP space. | `org:"Example Corporation"` |
| **`asn:`** | Scopes search to an Autonomous System Number (ASN). | `asn:AS15169` |
| **`net:`** | Restricts search to a specific CIDR IP subnet. | `net:198.51.100.0/24` |
| **`port:`** | Filters by listening network port. | `port:9200` or `port:3389` |
| **`http.title:`** | Searches the HTML `<title>` tag of web services. | `http.title:"Dashboard" ssl:"example.com"` |
| **`http.html:`** | Searches within the HTML body response. | `http.html:"/actuator/health"` |
| **`http.status:`** | Filters by HTTP response status code. | `http.status:200 port:8080` |
| **`product:`** | Filters by software or service name identified by Shodan. | `product:"MySQL"` or `product:"Elasticsearch"` |
| **`country:`** | Filters by two-letter ISO country code. | `country:"US"` or `country:"IN"` |

---

## 💡 High-Value Bug Bounty Discovery Strategies

### 1. SSL Certificate Pivoting (Uncovering Origin IP Behind Cloudflare)
When an application uses a CDN or WAF (e.g. Cloudflare), requests to the domain name are proxied through CDN edge nodes. If the origin server directly exposes its own SSL certificate on its public IP:
```text
ssl.cert.subject.CN:"example.com" -http.title:"Cloudflare"
ssl:"example.com" 200
```
*Impact:* Uncovers the origin server's direct IP address. If the origin accepts direct connections, the WAF and CDN rate limits can be completely bypassed.

---

### 2. Exposed Database & In-Memory Cache Instances
Search for database services listening on target company hostnames or IP ranges:
```text
hostname:"example.com" "product:MySQL"
hostname:"example.com" "product:MongoDB"
hostname:"example.com" port:9200 json
hostname:"example.com" "product:Redis"
hostname:"example.com" "port:5432 PostgreSQL"
```

---

### 3. CI/CD Controllers & Monitoring Dashboards
Locate internal DevOps infrastructure exposed to the public internet:
```text
ssl:"example.com" http.title:"Jenkins"
ssl:"example.com" "X-Jenkins" port:8080
ssl:"example.com" http.title:"Grafana"
ssl:"example.com" http.title:"Kibana"
ssl:"example.com" http.title:"Prometheus Time Series Collection"
```

---

### 4. Spring Boot Actuators & Microservice Health
```text
ssl:"example.com" http.html:"/actuator/health"
ssl:"example.com" http.title:"Swagger UI"
ssl:"example.com" http.title:"Eureka"
```

---

### 5. VPN Gateways & Remote Access Endpoints
```text
ssl:"example.com" http.html:"/dana-na" (Pulse Secure VPN)
ssl:"example.com" http.html:"/remote/login" (Fortinet VPN)
hostname:"example.com" port:3389 "Remote Desktop"
```

---

## 🛠️ Safe Verification & Reporting Workflow

1. **Verify Scope Membership:** Cross-reference discovered IP addresses with the program's written in-scope CIDR blocks or wildcard domain list.
2. **Do Not Exploit Open Ports:** If an open Redis or Elasticsearch instance is found, **do not dump keys or read database indexes**.
3. **Safe PoC:** Send a single curl command to retrieve the service banner:
   ```bash
   curl -I https://[DISCOVERED_IP]:8443
   ```
4. **Report Responsibly:** Provide the Shodan query, the affected IP/port, and the specific exposure risk to the security team.

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Restrict Management Ports via Cloud Security Groups
Never expose administrative or database ports (22, 3306, 5432, 6379, 9200, 3389) directly to `0.0.0.0/0`. Restrict ingress strictly to corporate VPN IPs or use AWS PrivateLink / VPC peering.

### 2. Restrict Direct Origin IP Access (Cloudflare Authenticated Origin Pulls)
Configure origin firewalls to drop all connections that do not originate from authenticated CDN IP ranges:
```nginx
# Nginx: Only allow Cloudflare IP ranges
allow 173.245.48.0/20;
allow 103.21.244.0/22;
# ... (Cloudflare IP blocks)
deny all;
```

### 3. Bind Services to Localhost (`127.0.0.1`)
Ensure databases and internal microservices bind only to the local loopback interface or private VPC interfaces, never public addresses.

---

## 📊 CVSS 3.1 Severity Reference

- **Publicly Exposed Unauthenticated Database (Elasticsearch, Redis):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` (Score: **9.1 - Critical**)
- **Origin IP Disclosure (WAF / Rate-Limit Bypass):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:N` (Score: **6.5 - Medium**)
- **Exposed VPN Gateway with Known Unpatched CVE:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` (Score: **10.0 - Critical**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Discovered asset is verified to belong to target scope.
- [ ] Confirmed service exposure passively via Shodan query.
- [ ] Verified reachability with minimal non-destructive HTTP/banner probe.
- [ ] Did not attempt unauthenticated database reads or credential stuffing.
- [ ] Captured IP, port, service banner, and Shodan query in report.
