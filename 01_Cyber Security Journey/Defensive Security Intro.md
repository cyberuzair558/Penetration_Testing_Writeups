# 🔵 Defensive Security Intro — TryHackMe Writeup

![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen) ![Category](https://img.shields.io/badge/Category-Defensive%20Security-blue) ![Room](https://img.shields.io/badge/TryHackMe-Room-purple)

**Room Link:** [tryhackme.com/room/defensivesecurityintro](https://tryhackme.com/room/defensivesecurityintro)

---

## 📖 Overview

While offensive security focuses on breaking in, **Defensive Security** is the counterpart — preventing intrusions from happening, and detecting and responding to them when they do. This room covers the core pillars of the Blue Team world: **SOC operations, Threat Intelligence, DFIR, and Malware Analysis**, followed by a simulated SIEM investigation.

---

## 🎯 Objective

Understand the foundational concepts and responsibilities of a defensive security team, and gain hands-on exposure to how a SOC analyst investigates alerts using a simulated SIEM environment.

---

## 🧩 Core Concepts Covered

### 🏢 Security Operations Center (SOC)
A SOC is a team of security professionals responsible for continuously monitoring an organization's network and systems to detect malicious activity. Key areas of focus include:

- **Vulnerabilities** — weaknesses that need patching or mitigation
- **Policy violations** — actions that break internal security rules
- **Unauthorized activity** — e.g. use of stolen credentials
- **Network intrusions** — active breaches requiring rapid detection

### 🧠 Threat Intelligence
The practice of collecting and analyzing information about potential adversaries — their tactics, techniques, and motives — to build a **threat-informed defense**. Different organizations face different threat actors (e.g. nation-state groups vs. financially motivated ransomware gangs), so intelligence is tailored to the specific risk profile of the business.

### 🕵️ Digital Forensics & Incident Response (DFIR)

**Digital Forensics** investigates attack evidence across:
| Source | What It Reveals |
|---|---|
| File System | Installed programs, created/deleted/modified files |
| System Memory | In-memory malware that never touched disk |
| System Logs | Activity history, even after attacker cleanup attempts |
| Network Logs | Packet-level evidence of attacker communication |

**Incident Response** follows a structured 4-phase lifecycle:

1. **Preparation** — building a trained team and preventative controls
2. **Detection & Analysis** — identifying and assessing the severity of an incident
3. **Containment, Eradication & Recovery** — stopping the spread, removing the threat, restoring systems
4. **Post-Incident Activity** — documenting lessons learned to prevent recurrence

### 🦠 Malware Analysis
Understanding malicious software through two techniques:
- **Static Analysis** — examining the malware's code without executing it (requires assembly-level knowledge)
- **Dynamic Analysis** — running the malware in a sandboxed environment to observe its real behavior

Common malware types covered: **Viruses**, **Trojan Horses**, and **Ransomware**.

---

## 🖥️ Hands-On: Simulated SOC Investigation

The practical portion placed me in the role of a **SOC Analyst** at a bank, using a simulated **SIEM (Security Information and Event Management)** dashboard — the kind of tool real analysts use to aggregate alerts from across an organization's infrastructure.

**Scenario tasks included:**
- Reviewing generated security alerts (e.g. failed login attempts, connections from unknown IPs)
- Applying analyst judgment to distinguish **false positives** from genuine threats
- Investigating the simulation step-by-step to trace suspicious activity to its source

This exercise highlighted a critical SOC skill: **not every alert is malicious**, and part of the analyst's job is triaging noise from real threats efficiently and accurately.

---

## 💡 Key Takeaway

Defensive security is less about a single tool and more about **process and vigilance** — combining people (SOC analysts), intelligence (threat data), and structured response plans (IR lifecycle) to protect an organization continuously. Understanding this side of security gives crucial context for offensive work too: knowing how defenders think makes for a better, more responsible attacker.

---

## 🧠 Skills Practiced

`SOC Fundamentals` `Threat Intelligence` `DFIR` `Malware Analysis Basics` `SIEM Alert Triage` `Incident Response Lifecycle`

---

## ✅ Status: Completed

*This writeup documents my personal learning process. TryHackMe answers/flags are intentionally omitted — this is a walkthrough of concepts and methodology, not a copy-paste solution.*
