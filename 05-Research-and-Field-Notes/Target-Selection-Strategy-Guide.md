# Target Selection Strategy Guide for Bug Bounty Beginners

## 1. Executive Summary & The Beginner Paradox
The #1 reason beginners give up on bug bounty hunting is **not** a lack of hacking knowledge—it is **poor target selection**. 

Beginners often pick the most famous companies (e.g., Google, Apple, Meta, Uber, Starbucks, or long-standing public programs) that have been audited by thousands of senior researchers and continuous automated pipelines for 5–10 years. Encountering 50 duplicates or finding zero bugs leads to burnout.

Target selection is an analytical skill. By filtering for **program age, scope depth, asset complexity, and triage responsiveness**, beginners can shift from high-competition dead ends to high-probability hunting grounds.

```
+---------------------------------------------------------------------------------+
|                           BEGINNER TARGET PIPELINE                              |
+---------------------------------------------------------------------------------+
                                       |
    [Stage 1: Avoid Hardened Giants]  ---> Skip 5-year-old top-tier programs
                                       |
    [Stage 2: VDP / Fresh Programs]   ---> Hunt for 5-10 valid triage reports
                                       |
    [Stage 3: Unlock Private Invites] ---> Private programs (90% less competition)
                                       |
    [Stage 4: Feature-Rich SaaS]      ---> Multi-tenant apps, complex role trees
```

---

## 2. The Four Fatal Target Traps

| Trap | What Beginners Do | Why It Fails | What To Do Instead |
| :--- | :--- | :--- | :--- |
| **1. The Big-Brand Trap** | Jump immediately into Fortune 500 public programs. | Millions of eyes; low-hanging fruit patched years ago; massive duplicate rate. | Target recently launched programs (< 60 days old) or newly added assets. |
| **2. The Marketing Page Trap** | Testing static landing pages (`www.target.com`). | Static blogs, CDN assets, and marketing blurbs have almost zero dynamic endpoints. | Hunt on core web apps: `app.target.com`, `dashboard.*`, `portal.*`. |
| **3. The Zombie Program Trap** | Submitting to programs with no activity for 6+ months. | Reports sit unresolved for 180+ days; team rarely awards bounties or triages. | Check the program's **"Response Efficiency"** and **"Recent Bounties Awarded"**. |
| **4. The Scanner-Only Trap** | Running automated Nuclei/Nmap scans on public scopes. | Top hunters run cloud scanners within 5 seconds of scope release. | Focus on **business logic, multi-user access control, and manual workflow testing**. |

---

## 3. The Target Selection Decision Matrix

```
                      +-----------------------------+
                      | What is your primary skill? |
                      +-----------------------------+
                                     |
               +---------------------+---------------------+
               |                                           |
        [Deep Web Testing]                          [Recon & Assets]
               |                                           |
               v                                           v
    +-----------------------+                   +-----------------------+
    | Pick Narrow Scope     |                   | Pick Wide Scope       |
    | (Single SaaS App)     |                   | (*.company.com)       |
    +-----------------------+                   +-----------------------+
               |                                           |
       Key Characteristics:                        Key Characteristics:
       - Multi-tenant organizations                - 50+ subdomains
       - Multiple user roles (Admin/Member)        - Diverse technology stacks
       - Complex CRUD operations                   - Staging/dev environments
       - File processing & webhooks                - API documentation exposed
```

---

## 4. How to Evaluate Program Health on Platforms

Before testing any program on HackerOne, Bugcrowd, Intigriti, or YesWeHack, check these metrics:

### 1. Response & Triage Speed
- **Ideal Metric**: First response within 24–48 hours, resolution within 2–4 weeks.
- **Red Flag**: "Average time to first response: 2 months" or "Reports unresolved: > 50%".

### 2. Minimum & Average Bounty
- Programs with high average payouts often have dedicated security engineers who value edge-case logic bugs.
- If testing VDP (points only), prioritize programs that give **HackerOne Reputation points** or **swag/hall of fame**, which unlock private program invites.

### 3. Scope Clarity & Safe Harbor
- Look for **"Gold Standard Safe Harbor"** or **"Full Safe Harbor"** to ensure legal protection.
- Ensure out-of-scope vulnerabilities (e.g., self-XSS, SPF/DMARC records, rate-limiting on contact forms) are clearly listed so you don't waste time on non-eligible findings.

---

## 5. Feature-Rich Target Checklist (Where Bugs Actually Live)

High-probability targets have applications with **deep state transitions and interactive workflows**. Use this checklist when selecting an application to hunt:

```markdown
- [ ] User Roles: Does the app offer Role-Based Access Control? (Owner, Admin, Manager, Guest, Read-Only)
- [ ] Team / Multi-Tenancy: Can users invite collaborators or switch organizations?
- [ ] File Uploads: Does the app parse avatars, PDF reports, XML, CSV, or SVGs?
- [ ] Integrations & Webhooks: Does it connect to Slack, GitHub, Google Drive, or Stripe?
- [ ] Payment / Subscriptions: Are there tier upgrades, promo code fields, or invoice downloads?
- [ ] Data Export: Does it allow exporting CSV, PDF, or Excel sheets (potential CSV Injection / SSRF)?
- [ ] API Architecture: Are REST or GraphQL endpoints exposed with parameter-driven queries?
```

> [!TIP]
> If a web application only has a contact form and a newsletter signup, **skip it immediately**. Move to an app where users create, share, edit, and delete complex business resources.

---

## 6. The "Fresh Code" Advantage

Vulnerabilities are introduced when **developers push new code**. You can gain an edge over established hunters by tracking updates:

1. **Watch Scope Expansion Announcements**: When an existing program adds a new subdomain or mobile app to scope, focus 100% of your attention there for the first 48 hours.
2. **Monitor Company Changelogs**: Look for blog posts announcing "New Feature: Team Workspaces" or "Introducing Webhook Notifications". Test the new features on day 1.
3. **Inspect Subdomain Certificate Transparency Logs**: Subdomains created within the last 7 days (`crt.sh`, `subfinder`) frequently lack complete WAF rules or authentication controls.
4. **Mobile App Releases**: Decompile newly updated Android APKs (`jadx-gui`) to discover unreleased staging API endpoints (`api-v2-dev.target.com`).

---

## 7. Recommended Target Progression for Beginners

```
Month 1: Fundamentals & Proof-of-Triage
├── Join 3 active VDP programs with wide scopes (*.target.com)
├── Focus exclusively on IDORs and Access Control bypasses
└── Objective: 3 to 5 triaged reports, build HackerOne/Bugcrowd reputation

Month 2: Feature-Rich Paid Programs
├── Filter for newly launched programs (< 30 days old)
├── Pick 2 SaaS targets and spend 1-2 weeks understanding business logic
└── Objective: Find your first valid P3/P2 vulnerability and earn first bounty

Month 3+: Private Programs & Specialization
├── Leverage platform reputation to accept private program invites
├── Choose a niche: GraphQL security, OAuth implementation flaws, or Cloud/SSRF
└── Objective: Consistent hunting on low-noise private assets
```

---

## 8. Summary Checklist for Evaluating Any Target

- [ ] Is this target less than 1 year old, or has it updated scope recently?
- [ ] Is the triage time under 7 days?
- [ ] Does the app have at least 2 distinct user permission levels?
- [ ] Are you testing an interactive workflow rather than a static landing page?
- [ ] Do you have two distinct test accounts created for dual-session testing?
