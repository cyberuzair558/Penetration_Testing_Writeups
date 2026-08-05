![Status](https://img.shields.io/badge/Status-Complete-brightgreen) ![Focus](https://img.shields.io/badge/Focus-PenTest%2B%20%2F%20Ethical%20Hacking-0e7c7b) ![Module](https://img.shields.io/badge/Module-2%20%7C%20Section%202.2-1b2a4a)

# 🎯 Section 2.2 — Scoping & Organizational / Customer Requirements

> Notes from Cisco Ethical Hacking / PenTest+ style coursework — Module 2.
> Companion to [Section 2.1 (GRC) notes].

## 📑 Table of Contents
- [✅ Coverage Checklist](#-coverage-checklist)
- [⚡ At a Glance](#-at-a-glance)
- [2.2.1 — Why Scoping Matters](#221--why-scoping-matters)
- [2.2.2 — Rules of Engagement (RoE)](#222--rules-of-engagement-roe)
- [2.2.4 — Target List and In-Scope Assets](#224--target-list-and-in-scope-assets)
- [2.2.6 — Validating the Scope of Engagement](#226--validating-the-scope-of-engagement)
- [2.2.7 — Known vs. Unknown Environment Testing](#227--strategy-unknown-vs-known-environment-testing)
- [🧪 Applied Lab — Nexus Plaza Case Study](#-applied-lab--pre-engagement-scope--planning)
- [📋 Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [💼 Interview Q&A](#-interview-questions-recruiters-actually-ask)

---

## ✅ Coverage Checklist
- [x] 2.2.1 — Why Scoping Matters (Foundation Recap)
- [x] 2.2.2 — Rules of Engagement (RoE)
- [x] 2.2.4 — Target List and In-Scope Assets
- [x] 2.2.6 — Validating the Scope of Engagement
- [x] 2.2.7 — Strategy: Unknown vs. Known Environment Testing
- [x] Applied Lab — Pre-Engagement Scope & Planning (Nexus Plaza)

---

## ⚡ At a Glance
> - **Scoping** = defining exactly WHAT is tested, WHAT is not, and HOW.
> - **RoE** = the operational rulebook (timing, comms, IPs, allowed techniques).
> - Poor scoping is the **#1 cause of legal exposure** and client conflict in real engagements.
> - **Scope creep** silently expands work without payment or authorization — the #1 margin-killer for security firms.
> - **Known vs. Unknown environment testing** decides cost, time, and realism of the whole engagement.

---

## 2.2.1 — Why Scoping Matters

Scoping is the single most important pre-engagement task. It determines exactly which systems, networks, and applications are authorized for testing — and just as importantly, which ones are **not**. Without a clear scope, a tester can accidentally attack a third-party system, disrupt production services, or breach client trust — all with real legal and financial consequences.

**Core principle:** the broader the scope, the greater the skill set, time, and resources required from the testing team. Scope size directly drives team composition, pricing, and timeline.

> 💡 **Recruiter/Interview Angle:** Scoping shows you understand *risk management*, not just tools. This is where the junior-vs-senior pentester distinction is made — juniors run tools, seniors define boundaries.

---

## 2.2.2 — Rules of Engagement (RoE)

The RoE is the **operational rulebook** of the assessment. Where the SOW defines *what* work happens and its cost, the RoE defines *how* testing is actually conducted, day to day.

> **SOW vs RoE — don't confuse them**
> - SOW = business-level document (what, when, cost)
> - RoE = operational/technical-level document (how, under what conditions)
> - Both are usually agreed before Day 1 and complement each other

### Core Elements of an RoE

| Element | What It Defines | Why It Matters |
|---|---|---|
| Testing Timeline | Total duration (e.g., 3 weeks, via Gantt chart) | Sets client expectations & resourcing |
| Location of Testing | Where testing physically/remotely originates | Relevant for on-site/physical engagements |
| Time Window | Hours of testing (e.g., 9 AM–5 PM EST) | Limits business disruption risk |
| Communication Method | Weekly meetings, final report, email updates | Avoids confusion during engagement |
| Security Controls Awareness | IPS, firewalls, DLP already in place | Prevents false alarms / SOC triggers |
| Source IP Addresses | IPs testing will originate from (whitelisting) | Client distinguishes tester traffic from real attacks |
| Allowed/Disallowed Tests | e.g., no SQLi on production, no social engineering | Protects live systems and data |

**Project management tools:** Gantt Chart / Work Breakdown Structure (WBS) visually map every phase.
Example: `Pre-engagement (6d) → App 1 Testing (15d) → App 2 Testing (15d) → Report Delivery (1d) → Knowledge Transfer (3d)`

<details>
<summary>🔑 Permission to Test / Permission to Attack (click to expand)</summary>

The tester's **"get out of jail free"** document:
- Formal written authorization proving the activity is legal and sanctioned
- Can be standalone or bundled into the main contract
- Must be produced immediately if law enforcement or a security team intervenes mid-test

</details>

---

## 2.2.4 — Target List and In-Scope Assets

Once RoE conditions are set, the next step is precisely documenting **which assets** are in scope — networks, wireless SSIDs, APIs, applications — down to IP ranges and domain names.

### API Documentation Formats

| Format | Type | Key Detail |
|---|---|---|
| SOAP | XML-based | Governed by XSD (XML Schema Definition) specs |
| Swagger / OpenAPI | REST-based | Now the OpenAPI Specification (OAS); maps all endpoints |
| WSDL | XML-based | Web Services Description Language |
| GraphQL | Query language + runtime | Modern REST alternative; executes against a type system |
| WADL | XML-based | Web Application Description Language |

### Additional Client-Provided Resources
- **SDK** — tools to interact with a specific framework, OS, or hardware platform
- **Source Code** — enables White-Box / Known-Environment depth
- **Sample App Requests** — captured via Burp Suite or OWASP ZAP
- **Architecture Diagrams** — visual map of what's in scope

### Must Be Documented Before Testing
- Physical location of testing
- DNS / FQDNs allowed
- **External** (internet-facing) vs **Internal** (insider-threat simulation) testing
- **First-party** hosted vs **Third-party** hosted apps — cloud providers (AWS, etc.) have their **own** pentest policies

### ⚠️ Scope Creep — The #1 Risk

> Scope creep = the **uncontrolled, silent growth** of a project's scope beyond what was originally agreed (aka *kitchen sink syndrome*, *requirement creep*, *function creep*).

- Many security firms fail because they don't recognize scope creep in time
- It consumes unpaid resources and creates unauthorized (legally risky) testing
- **Professional response:** any new request goes through a formal **Change Request** or **Amended SOW** — never expand scope verbally

---

## 2.2.6 — Validating the Scope of Engagement

Validating scope means confirming it with the client, reviewing all contracts, and understanding who the final report audience is.

### Key Questions to Identify Report Audience
1. Why does this entity/individual need the report?
2. What position does the primary recipient hold (ISM, CISO, CIO, CTO, Technical Team)?
3. What is the ultimate purpose/goal of the report?
4. Who has the authority to act on the findings?
5. Who will have access to the report (need-to-know basis)?

### Stakeholder & Emergency Contact Documentation

| Field | Details Captured |
|---|---|
| Primary Stakeholder | Name, Title, Email, Responsibility, Phone(s), Address |
| Primary Emergency Contact | Phone, Email, Address |
| Secondary Emergency Contact | Phone, Email, Address |
| Communication Cadence | How often (daily/weekly) and by what channel |

### Secure Data Transfer
- **File transfer:** SCP (Secure Copy Protocol), SFTP (Secure File Transfer Protocol)
- **Encrypted email:** PGP (Pretty Good Privacy), S/MIME

> Findings must never travel over unencrypted channels — an intercepted report can expose the entire client environment.

<details>
<summary>💰 Budget & ROI Conversation (click to expand)</summary>

Common business-side questions every pentester should expect:
- *"Why do we need this if we already have firewalls/IPS?"* → Automated tools catch known threats; humans simulate creative, real attacker behavior.
- *"How do I justify this cost to my boss?"* → Tie findings to business risk, not just technical severity.
- *"What's our ROI?"* → Frame around breach cost avoidance, compliance mandate, and reputational risk.

</details>

### 📌 Point-in-Time Assessment — Core Insight

A pentest reflects security posture at **one moment in time**. It is never a permanent guarantee.

| Engagement | Systems Tested | Critical | High | Medium | Low |
|---|---|---|---|---|---|
| Pen Test 1 (Year 0) | 1,000 | 5 | 11 | 32 | 45 |
| Pen Test 2 (Year 1) | 1,100 | 3 | 31 | 10 | 7 |
| Pen Test 3 (Year 2) | 2,200 | 15 | 22 | 8 | 15 |

> **Read the numbers in context:** raw critical-vuln count rose 5 → 15, but systems tested more than *doubled* (1,000 → 2,200). Proportionally, security may actually be improving — always normalize findings against scope size. Ask: are these tests happening just to check a compliance box, or to genuinely improve security?

**Tester's real responsibility:**
- Provide clear, achievable mitigation strategies — not just a list of problems
- Perform impact analysis — translate findings into business risk
- Agree on remediation timelines with stakeholders

---

## 2.2.7 — Strategy: Unknown vs. Known Environment Testing

> **Terminology update:** Black-Box → **Unknown-Environment Testing** · White-Box → **Known-Environment Testing** (both old and new terms still appear in interviews)

| Approach | Information Given | Perspective | Typical Scope | Best For |
|---|---|---|---|---|
| **Unknown-Environment** (Black-Box) | Minimal — only domains/IPs | External attacker | Broad — internal audit, endpoint scanning | Realistic attack simulation, defense/detection exercise |
| **Known-Environment** (White-Box) | Extensive — diagrams, configs, credentials, source code | Insider/full-knowledge | Narrow — find one path in, stop | Maximum depth, fast & thorough results |

**Grey-Box (middle ground):** a partial-info hybrid — client shares limited detail about a specific concern, reducing time/cost while still reaching meaningful results.

> ⏱️ **Deciding factor: Time & Money.** Unknown-environment testing is more realistic but slower/costlier. Known-environment testing is faster/cheaper but less realistic. Course guidance: modern adversaries are sophisticated enough that unknown-environment testing is often the better default when budget allows.

---

## 🧪 Applied Lab — Pre-Engagement Scope & Planning

**Case Study: Nexus Plaza** (online retail company). Task: help the lead auditor scope the engagement by combining network topology with a CEO/IT Director interview.

### Network Topology

| Zone | Contents |
|---|---|
| LAN | Admin, Warehouse, Finance, Customer Service, Shipping, IT |
| DMZ | Internet-facing servers |
| Data Center (Houston) | 25 servers — Administration, Amazon Support, Operations, Logistics + SQL Server + SAN |

### Turning Interview Statements Into Scope Decisions

| What the Client Said | Scoping Impact |
|---|---|
| "Competitor hit by ransomware — worried about warehouse/shipping" | Defines the #1 risk driver & test focus |
| "No testing on the Amazon storefront data" | Explicit out-of-scope boundary |
| "Test Operations and Logistics clusters" | In-scope systems |
| "Access via isolated VLAN" | Defines source IPs for testing |
| "Use the development SQL Server, not production" | Invasive tests confined to dev |
| "Maintenance window 2–6 AM Fri–Sun" | Time window for disruptive tests |
| "Only warehouse/operations staff emails provided" | Social engineering scope, explicitly limited |
| "End users won't be told testing is happening" | Unknown-environment approach for realism |
| "We have firewall + IDS" | Security controls that may flag tester traffic |
| "Weekly report + teleconference" | Communication method & frequency |

<details>
<summary>📋 Filled Scope Worksheet — key answers (click to expand)</summary>

- **Biggest concern:** Ransomware risk to inventory/shipping systems
- **In scope:** Operations & Logistics clusters (`172.26.0.0/21`, `172.27.0.0/21`) + MS SQL Server
- **Out of scope:** Administration, Amazon Support clusters, LAN ranges
- **Environment:** Mostly production; invasive SQL testing on dev only
- **Internal testing:** Yes, via isolated VLAN. End-user systems: not in scope
- **Social engineering:** Allowed, limited email list. **DoS:** Allowed, only in maintenance window
- **Wireless & web services:** Not in scope. **Employee awareness:** Only select managers/IT know
- **Data center location:** Houston

</details>

### Filled Rules of Engagement Table

| RoE Element | Nexus Plaza Answer |
|---|---|
| Timeline | Starts in 2 weeks, final report within 60 days |
| Location | Houston Facility |
| Time Window | Non-invasive: business hours; Invasive: 2–6 AM Fri–Sun |
| Communication | Reports + teleconferences |
| Security Controls | Firewalls and IDS already installed |
| Sensitive Data Handling | Signed NDA required |
| Source IPs | Internal IT Department VLAN |
| Allowed/Disallowed | Only Operations/Logistics servers; SQL only on dev; social engineering limited |
| Client Contacts | Warehouse Manager, IT Director, Operations Manager |

> 🎓 **Key lesson:** Clients rarely say "here is the scope" directly — scope is extracted from casual conversation. Every offhand statement is actually a hard scope boundary. The skill being tested is active listening + translating business language into RoE/scope elements.

---

## 📋 Quick Revision Cheat Sheet

| Concept | One-Line Recall |
|---|---|
| RoE vs SOW | RoE = how testing happens; SOW = what/when/cost |
| Scope Creep | Uncontrolled scope growth — always require a formal change request |
| Known-Environment | Full info given (aka White-Box) — fast, deep, less realistic |
| Unknown-Environment | Minimal info given (aka Black-Box) — realistic, slower, costlier |
| Grey-Box | Partial info — middle ground on time/cost/realism |
| Point-in-Time Assessment | A pentest reflects one moment only — not a permanent guarantee |
| PAN Rule (recall) | Presence of PAN is what triggers PCI DSS applicability |
| Secure Transfer | SCP/SFTP for files, PGP/S/MIME for encrypted email |
| Permission to Test | The tester's "get out of jail free" authorization document |
| Target List | IP ranges, SSIDs, APIs (SOAP/Swagger/WSDL/GraphQL/WADL), source code, diagrams |

---

## 💼 Interview Questions Recruiters Actually Ask

<details>
<summary><b>Q: What is the difference between black-box and white-box testing, and which would you recommend?</b></summary>

It depends on the client's goal, time, and budget. Unknown-environment (black-box) testing simulates a real external attacker with no prior knowledge — best for realistic detection exercises. Known-environment (white-box) testing gives full access to diagrams, credentials, and even source code, making it faster and more thorough but less realistic. If budget allows, unknown-environment testing tends to give more realistic insight since modern adversaries are highly sophisticated.
</details>

<details>
<summary><b>Q: What is scope creep and how do you handle it?</b></summary>

Scope creep is the uncontrolled, silent expansion of a project's scope beyond what was originally agreed. I handle it by never expanding testing verbally — any new request goes through a formal change request or amended SOW before any additional system is touched.
</details>

<details>
<summary><b>Q: What's the difference between an RoE and an SOW?</b></summary>

The SOW is a business-level document defining what work happens, the timeline, and cost. The RoE is an operational/technical document defining exactly how testing will be conducted — time windows, source IPs, allowed and disallowed techniques.
</details>

<details>
<summary><b>Q: What is a limitation of penetration testing?</b></summary>

A pentest is a point-in-time assessment — it reflects security posture only at the moment of testing. It cannot guarantee future security, since new vulnerabilities and systems appear constantly. It also can't be the sole indicator of overall security; mitigation guidance and remediation timelines matter as much as the findings themselves.
</details>

<details>
<summary><b>Q: How do you extract scope information from a client interview?</b></summary>

Clients rarely state scope explicitly. I listen for statements that imply boundaries — e.g., "don't touch the Amazon data" is an out-of-scope instruction, and "use the dev server for SQL testing" means invasive testing should avoid production. Each statement gets translated into a formal scope or RoE entry.
</details>

<details>
<summary><b>Q: Why is source IP whitelisting important during an engagement?</b></summary>

It lets the client's security team distinguish authorized tester traffic from a real attack, preventing false alarms and unnecessary incident response during a scheduled engagement.
</details>

---

<sub>📘 Part of an Ethical Hacking / PenTest+ study series — Module 2. See also: Section 2.1 (Governance, Risk & Compliance).</sub>
