# 🛡️ Ethical Hacking — Penetration Testing Notes
## Section 2.1 — Governance, Risk & Compliance (GRC)

> **Course context:** Cisco Ethical Hacking Course — Module 2, Section 2.1
> **Focus:** Legal & regulatory foundation every penetration tester needs before touching a client system.

![Status](https://img.shields.io/badge/Topics-9%2F9%20Complete-brightgreen)
![Focus](https://img.shields.io/badge/Focus-GRC%20%7C%20Legal%20%7C%20Compliance-blue)

---

## 📑 Table of Contents

- [At a Glance](#-at-a-glance)
- [2.1.1 — Overview](#211--overview)
- [2.1.2 — Regulatory Compliance Considerations](#212--regulatory-compliance-considerations)
  - [A. Export Controls — Wassenaar Arrangement](#a-export-controls--wassenaar-arrangement)
  - [B. Financial Sector Regulations](#b-financial-sector-regulations)
  - [C. Healthcare Sector — HIPAA](#c-healthcare-sector--hipaa)
  - [D. PCI DSS — Deep Dive](#d-pci-dss--deep-dive)
  - [E. Key Technical Elements Verified in Every Compliance Assessment](#e-key-technical-elements-verified-in-every-compliance-assessment)
- [2.1.3 — Local Restrictions](#213--local-restrictions)
- [2.1.5 — Legal Concepts](#215--legal-concepts)
- [2.1.6 — Contracts](#216--contracts)
- [2.1.7 — Disclaimers](#217--disclaimers)
- [🔁 Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [🎤 Frequently Asked Interview Questions](#-frequently-asked-interview-questions)
- [✅ Final Takeaway](#-final-takeaway)

---

## ✅ At a Glance

**Topics completed — 9/9**

| # | Topic |
|---|---|
| 2.1.1 | Overview |
| 2.1.2 | Regulatory Compliance Considerations |
| 2.1.3 | Local Restrictions |
| 2.1.4 | Practice — Regulations |
| 2.1.5 | Legal Concepts |
| 2.1.6 | Contracts |
| 2.1.7 | Disclaimers |
| 2.1.8 | Practice — Legal Concepts |
| 2.1.9 | Lab — Compliance Requirements |

> **What this section covers:** Section 2.1 builds the legal and regulatory foundation every penetration tester needs before touching a client system — identifying which laws/standards apply to a client's industry, understanding local and international legal restrictions, and knowing the contractual documents (SOW, NDA, MSA, RoE) that make an engagement legally authorized and professionally sound.

**Core skill demonstrated:** Ability to scope and legally ground a penetration testing engagement — mapping a client's industry to the correct compliance framework, identifying jurisdictional/legal restrictions, and structuring the pre-engagement paperwork (contracts, NDAs, disclaimers) that protects both the tester and the client.

---

## 2.1.1 — Overview

Before any technical testing begins, a penetration tester must understand the legal and business context surrounding the engagement. **Poor planning and scoping is the leading cause of legal exposure in this field.**

> 💡 **The Golden Rule:** Understand the client's industry, legal environment, and compliance obligations **first** — then design scope and rules of engagement strictly inside those boundaries.

---

## 2.1.2 — Regulatory Compliance Considerations

Different regulations apply depending on the client's industry and the type of data they handle. A tester must recognize which framework governs an engagement.

| Sector | Regulation | Key Requirement |
|---|---|---|
| Payment / Retail | **PCI DSS** | Secures card payment processing. Applicability driven by the **PAN** (Primary Account Number). |
| Healthcare | **HIPAA** | Protects ePHI via the Security Rule; applies to Providers, Health Plans, Clearinghouses, Business Associates. |
| US Financial | **GLBA** | Mandates periodic pentesting; broad definition of "financial institution"; enforced by FTC. |
| NY Financial | **NY DFS (23 NYCRR 500.05)** | Annual pentest + biannual vulnerability assessment (**mandatory**). |
| US Govt / Cloud | **FedRAMP** | Authorizes cloud service offerings for federal government use. |
| EU / Global | **GDPR** | EU citizens' control over personal data + cross-border export rules. |
| California, US | **CCPA** | State-level consumer privacy law. |

### A. Export Controls — Wassenaar Arrangement

40+ countries regulate export of conventional arms and dual-use goods/technologies. Certain security tools (encryption/cryptographic software) can be classified as "arms" and restricted by national law.

> ⚠️ **Real Risk:** Carrying advanced exploitation or encryption tools across borders for an engagement can be an export-control violation — a criminal offense — even without malicious intent. Some testers have been accused/arrested under laws like the US **Computer Fraud and Abuse Act (CFAA §1030(a)(5)(B))**.

### B. Financial Sector Regulations

- **GLBA (Title V, §501(b))** — broad "financial institution" definition (banks, payday lenders, mortgage brokers, auto lenders, tax preparers, debt collectors, etc.); mandates periodic pentesting; enforced by FTC.
- **FFIEC** — sets uniform examination standards for banking regulators.
- **FDIC Safeguards Act & FILs** — safeguard requirement letters.
- **NY DFS (23 NYCRR 500.05)** — annual pentest + biannual vulnerability assessment; mandatory.

### C. Healthcare Sector — HIPAA

HIPAA Security Rule (2003) protects **ePHI** (electronic Protected Health Information). Expanded by HITECH Act (2009), Breach Notification Rule (2009), and Omnibus Rule (2013).

| Covered Entity | Definition |
|---|---|
| **Healthcare Provider** | Doctors, clinics, hospitals, pharmacy, diagnostic imaging, etc. |
| **Health Plan** | Insurance companies, HMOs, Medicare/Medicaid, military/veterans programs. |
| **Healthcare Clearinghouse** | Converts nonstandard health info into standard format. |
| **Business Associate** | Entities creating/receiving/transmitting/accessing PHI on behalf of a covered entity. |

### D. PCI DSS — Deep Dive

**Key Roles**

| Role | Definition |
|---|---|
| **Merchant** | Accepts payment cards for goods/services; can also be a Service Provider. |
| **Acquirer** | "Acquiring bank" — maintains merchant relationships for card acceptance. |
| **Service Provider** | Processes/stores/transmits cardholder data on behalf of others (e.g., hosting, managed firewall). |
| **QSA** | Qualified Security Assessor — certified to conduct PCI DSS compliance assessments. |
| **ASV** | Approved Scanning Vendor — PCI SSC-approved external vulnerability scanning. |
| **PFI** | PCI Forensic Investigator — investigates/contains cardholder data breaches. |

**Account Data — What Must Be Protected**

| Cardholder Data (may be encrypted & stored) | Sensitive Authentication Data (NEVER stored post-auth) |
|---|---|
| Primary Account Number (PAN) — up to 19 digits | Full magnetic stripe / chip-equivalent data |
| Cardholder Name | CAV2 / CVC2 / CVV2 / CID |
| Expiration Date | PINs / PIN blocks |
| Service Code | — |

> 🔑 **The Golden Rule of PCI DSS:** PAN presence determines PCI DSS applicability. PAN must always be encrypted at rest. CVV/PIN/full track data must **never** be stored after authorization — finding this during a pentest is a critical, reportable issue.

- **CDE (Cardholder Data Environment):** the people, processes, and technology that handle cardholder/sensitive authentication data — scoping always starts by defining this boundary.
- **Luhn Algorithm:** validates card/ID numbers using modulo-10 math (Hans Peter Luhn, 1954; public domain).

### E. Key Technical Elements Verified in Every Compliance Assessment

| Element | What the Tester Verifies |
|---|---|
| **Data Isolation** | Network segmentation between the sensitive-data environment (e.g., CDE) and the general network — test by attempting to pivot across the boundary. |
| **Password Management** | No vendor default credentials; enforced length/complexity/session timeout; MFA in place. *(Most common real-world finding.)* |
| **Key Management** | Encryption keys not hardcoded, properly rotated, access-restricted. "A key is like a safe combination — the algorithm is worthless if the key leaks." *(Ref: NIST SP 800-57)* |

---

## 2.1.3 — Local Restrictions

Beyond industry regulation, testers must account for restrictions tied to geography, technical operations, and the client's own internal policy.

**1. Country-Specific Legal Restrictions**
- Pentesting laws vary by country — testers abroad must know local law before acting.
- Real risk example: testers accused/arrested under the US CFAA §1030(a)(5)(B).
- Always carry clear written authorization from the client proving permission to test.

**2. Technical / Operational Constraints**
- Tool restrictions dictated by the client.
- Operational limits — e.g., avoid SQL injection on a live production database (could corrupt it).
- Organization-specific tech, skill-set limits, known-exploit limits, and systems excluded due to criticality/performance issues.
- **Rule:** communicate all technical constraints with stakeholders before *and* during testing.

**3. Local Government / Privacy Requirements**
- **GDPR** (EU) and **CCPA** (California, US state law) — regional privacy laws that may apply on top of federal/industry regulation.

**4. Client's Own Corporate Policy**
- Clients may have internal vulnerability/pentesting policies defining restricted vs. nonrestricted systems.
- Always ask directly and document — don't assume the client has disclosed everything.

---

## 2.1.5 — Legal Concepts

Five foundational agreement types every pentester should recognize and expect on a professional engagement:

| Document | Purpose |
|---|---|
| **SLA** (Service-Level Agreement) | Documents min/max performance expectations — quality, timeline, cost. |
| **Confidentiality Agreement** | Defines how sensitive data (e.g., discovered passwords) is handled, who can access it, and requires deletion of all records after engagement. |
| **SOW** (Statement of Work) | Defines activities performed: timelines/report schedule, scope, location, technical requirements, payment schedule, tracked miscellaneous items. Can stand alone or be part of an MSA. |
| **MSA** (Master Service Agreement) | Reusable contract for recurring engagements with the same client — avoids renegotiating terms every time. |
| **NDA** (Non-Disclosure Agreement) | Legally protects confidential information. Types: **Unilateral** (one party discloses), **Bilateral/Mutual** (both share), **Multilateral** (3+ parties, e.g., business partners involved). |

---

## 2.1.6 — Contracts

The contract is the most important document in an engagement — it specifies payment terms and provides clear documentation of services. It must be **specific, unambiguous**, and ideally reviewed by a lawyer.

- Client may require collected data stay within the country tested.
- PII subject to specific laws may require written authorization or bound commitment before removal.
- **Signing authority:** must obtain signature from someone with proper legal authority; written authorization also needed from any impacted third parties (ISPs, cloud providers, business partners).

---

## 2.1.7 — Disclaimers

Standard protective statements added to pre-engagement documentation and the final report:

- **Point-in-time testing:** results reflect systems as of a stated date — new vulnerabilities are discovered daily; nothing is fully immune.
- **Report purpose:** report provides documentation only; the client decides remediation. Report does not protect against personal/business loss.
- **No warranties:** tester provides no warranty/certification that tested systems are fully compliant, defect-free, or compatible with all platforms.

---

## 🔁 Quick Revision Cheat Sheet

| Topic | One-Line Recall |
|---|---|
| GRC | Governance = internal rules; Risk = threats & impact; Compliance = meeting external laws. |
| PCI DSS | Applies wherever PAN is stored/processed/transmitted; CVV/PIN never stored. |
| HIPAA | Protects ePHI; 4 covered entities: Provider, Health Plan, Clearinghouse, Business Associate. |
| GLBA / NY DFS | GLBA mandates periodic pentest (broad institution definition); NY DFS = annual pentest + biannual VA. |
| Wassenaar Arrangement | 40+ countries control export of arms & dual-use tech; some security tools count as "arms." |
| Local Restrictions | Country law + technical constraints + local privacy law (GDPR/CCPA) + client's own policy. |
| SLA vs SOW vs MSA vs NDA | SLA = performance; SOW = what/when/where; MSA = reusable contract; NDA = confidentiality. |
| Contract Essentials | Specific, unambiguous, lawyer-reviewed, signed by proper authority, covers 3rd-party authorization. |
| Disclaimers | Point-in-time scope, report-is-documentation-only, no warranties. |

---

## 🎤 Frequently Asked Interview Questions

<details>
<summary><strong>Q1. Why must a pentester research the client's industry before scoping?</strong></summary>
<br>
Because the applicable regulation (PCI DSS, HIPAA, GLBA, etc.) and legal restrictions differ by industry and directly shape what can be tested, how, and how findings must be handled.
</details>

<details>
<summary><strong>Q2. What determines PCI DSS applicability?</strong></summary>
<br>
The presence of the PAN (Primary Account Number). If it isn't stored, processed, or transmitted, PCI DSS doesn't apply.
</details>

<details>
<summary><strong>Q3. Name the four HIPAA covered entities.</strong></summary>
<br>
Healthcare Provider, Health Plan, Healthcare Clearinghouse, and Business Associate.
</details>

<details>
<summary><strong>Q4. What's the difference between an SOW and an NDA?</strong></summary>
<br>
An SOW defines what work will be performed, when, and for how much. An NDA legally protects confidential information exchanged between the parties.
</details>

<details>
<summary><strong>Q5. Why is local/international law important in pentesting?</strong></summary>
<br>
Because pentesting laws vary by country, and testers have faced legal action (e.g., under the CFAA) for actions that were authorized by the client but violated local/national law.
</details>

<details>
<summary><strong>Q6. What must happen before removing PII discovered during testing?</strong></summary>
<br>
The tester must either commit to being bound by the relevant laws/regulations or obtain written authorization from the company before removing such data.
</details>

<details>
<summary><strong>Q7. Why include disclaimers in a pentest report?</strong></summary>
<br>
To limit liability — clarifying the test reflects a specific point in time, provides documentation only (not a guarantee), and includes no warranty of full compliance or compatibility.
</details>

---

## ✅ Final Takeaway

> A pentester's real skill isn't just running tools — it's knowing the client's industry, the laws that govern it, and the paperwork that authorizes and protects the engagement. Section 2.1 is the legal backbone that every technical chapter afterward depends on.

---

<p align="center"><sub>📘 Personal study notes — Cisco Ethical Hacking Course, Module 2, Section 2.1</sub></p>
