# 📁 Atlassian Jira — Security Review & Audit Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Auditing Atlassian Jira instances involves inspecting public user pickers, filters, dashboards, and REST APIs. Test strictly on authorized assets. **Never scrape internal corporate user lists, download sensitive client bug reports, or attempt exploitation of Jira server infrastructure.** Report confirmed permission misconfigurations responsibly.

---

## 📌 Executive Summary

**Atlassian Jira** is the industry standard enterprise issue tracking and project management platform. Because Jira houses proprietary software development tickets, infrastructure bug reports, security disclosures, and employee rosters, a misconfigured Jira instance presents an exceptionally high-value target for reconnaissance and intelligence gathering.

The most common vulnerabilities in Jira stem from **permissive default sharing settings** (where filters and dashboards are shared with "Everyone" on the internet) and **unauthenticated REST API endpoints**.

```
+---------------------------------------------------------------------------------+
|                         Atlassian Jira Attack Surface                           |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  User Enumeration APIs          Public Dashboards & Filters        Unauthenticated REST
  - /rest/api/2/user/picker      - "Shared with Everyone"           - /rest/api/2/project
  - /ViewUserHover.jspa          - Leaks tickets, sprints, bugs     - Gadget SSRF endpoints
```

---

## 🔍 Core Discovery & Testing Methodology

### 1. Public User Enumeration APIs
Jira instances frequently allow unauthenticated visitors to query internal user rosters:

#### Endpoint 1: User Picker API
```http
GET /rest/api/2/user/picker?query=a&maxResults=50 HTTP/1.1
Host: jira.example.com
```
*Vulnerable Indicator:* Returns a JSON array containing full employee names, usernames, and email addresses (`{"users": [{"name": "jdoe", "displayName": "John Doe", ...}]}`).

#### Endpoint 2: ViewUserHover Endpoint
```http
GET /secure/ViewUserHover.jspa?username=admin HTTP/1.1
Host: jira.example.com
```
*Test observation:* Does the server return user profile cards confirming the presence of usernames?

---

### 2. Publicly Shared Filters and Dashboards
In many self-hosted and cloud Jira deployments, when an employee creates a filter or dashboard, the sharing permission defaults to **"Public"** (which means completely public to the unauthenticated internet, not just internal company staff!):

#### Endpoints to Audit:
- **Public Filters:** `/ManageFilters.jspa?filterView=popular`
- **Public Dashboards:** `/ConfigurePortalPages.jspa?view=popular`
- **Search Request API:** `/sr/jira.issueviews:searchrequest-xml/temp/SearchRequest.xml?jqlQuery=`

#### Information Leaked by Public Filters:
- Internal project names and issue keys (`SEC-104`, `FINANCE-921`).
- Titles of proprietary features, security bug reports, and employee performance notes.
- Sprints, sprint goals, and development milestones.

---

### 3. Unauthenticated REST APIs
Jira exposes an extensive REST API under `/rest/api/2/`:

| API Endpoint | Potential Information Disclosure |
| :--- | :--- |
| `/rest/api/2/project` | Complete list of internal software projects, leads, and descriptions |
| `/rest/api/2/dashboard` | Internal dashboard cards and analytics metrics |
| `/rest/api/2/priority` | Custom issue priority levels |
| `/rest/api/2/status` | Workflow status names |
| `/rest/api/2/resolution` | Resolution types |

---

### 4. Gadget SSRF (`makeRequest`)
In legacy Jira Server versions (pre-8.4.0), the OpenSocial gadget specification implemented a proxy servlet:
```http
GET /plugins/servlet/gadgets/makeRequest?url=http://169.254.169.254/latest/meta-data/ HTTP/1.1
Host: jira.example.com
X-Atlassian-Token: no-check
```
If unpatched, this endpoint enables **Server-Side Request Forgery (SSRF)** against cloud metadata services or internal corporate intranets.

---

## 🛠️ Safe Educational Verification Template

### Safe HTTP Request (User Picker Probe)
```http
GET /rest/api/2/user/picker?query=test&maxResults=5 HTTP/1.1
Host: jira.example.com
User-Agent: Mozilla/5.0
```

### Vulnerable Response Body
```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "users": [
    {
      "name": "tester",
      "displayName": "QA Test Account",
      "active": true
    }
  ],
  "total": 1
}
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Disable Public Sharing Globally (Global Permissions)
In Jira Administration:
1. Navigate to **Jira Settings -> System -> Global Permissions**.
2. Remove the **"Anyone on the web"** / **"Public"** group from the **"Create Shared Objects"** permission.
3. Restrict sharing strictly to authenticated users or specific groups.

### 2. Restrict Unauthenticated Access to REST APIs
Block unauthenticated access to `/rest/api/2/user/picker` and `/rest/api/2/project` by enforcing authentication on reverse proxies or configuring Atlassian Access.

### 3. Audit Existing Public Filters & Dashboards
Run Atlassian's permission audit script to convert existing public filters to private or organization-scoped:
- In **System -> Shared Filters**, filter by shared with "Public" and update owners.

---

## 📊 CVSS 3.1 Severity Reference

- **Public Filter / Dashboard Leakage disclosing Security Tickets:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (Score: **7.5 - High**)
- **Unauthenticated Employee Email / User Enumeration:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` (Score: **5.3 - Medium**)
- **Gadget SSRF (Internal Network Pivot):** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N` (Score: **8.6 - High**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Target Jira instance is confirmed in program scope.
- [ ] Confirmed whether `/rest/api/2/user/picker` returns user records unauthenticated.
- [ ] Checked for publicly exposed filters under `ManageFilters.jspa`.
- [ ] Sampled only minimal non-sensitive records (under 3 items) as proof.
- [ ] Redacted all employee names, project keys, and ticket titles in report.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice evaluating Jira instances for information leaks and API exposure:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **TryHackMe** | [Jira CVE-2019-8451 & API Auditing](https://tryhackme.com/room/jira) | Medium | SSRF and unauthenticated user enumeration on REST endpoints |
| **Hack The Box** | [HTB Machine: BoardLight](https://app.hackthebox.com/machines/BoardLight) | Easy | Exploiting open project management boards and unauthenticated APIs |
| **PentesterLab** | [Jira Misconfiguration & Public Filters](https://pentesterlab.com/) | Medium | Discovering leaked customer tickets and employee PII |
