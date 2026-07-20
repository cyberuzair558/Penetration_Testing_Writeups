# 🧪 Nmap Post Port Scanning — TryHackMe Writeup

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Network%20Scanning-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)

> **Rooms:** Nmap Post Port Scans (Premium) — TryHackMe *(notes based on official Nmap documentation)*
> **Author of writeup:** *Your Name Here*
> **Purpose:** Documenting what happens **after** ports are discovered — using the Nmap Scripting Engine (NSE) to find real vulnerabilities, saving results professionally for reporting, and understanding (at a conceptual level) firewall/IDS evasion techniques.

---

## 📑 Table of Contents

- [Introduction](#-introduction)
- [NSE — Nmap Scripting Engine](#-nse--nmap-scripting-engine)
- [Output — Saving Results for Reporting](#-output--saving-results-for-reporting)
- [Firewall / IDS Evasion](#-firewall--ids-evasion)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 📝 Introduction

Finding open ports is only half the job. The real value in a penetration test comes from **what you do with that information afterward**: identifying actual vulnerabilities on those ports, and documenting everything in a way a client (or your own future self) can actually use.

This writeup covers three post-port-scan areas — **NSE, Output, and Firewall/IDS Evasion** — ranked by how relevant each is for entry-level pentesting and technical interviews. NSE and Output are must-know, hands-on skills. Firewall evasion is covered only at a conceptual level here, since it belongs to more advanced/red-team engagements.

---

## 🧠 NSE — Nmap Scripting Engine

### The Core Idea

NSE takes Nmap beyond "is this port open or closed?" and pushes it toward **"what's actually wrong with what's running on this port?"**

> 🚪 **Analogy:** Basic port scanning finds all the doors (ports) in a house. NSE tells you *which door has a broken lock* — i.e., which service is vulnerable and can actually be exploited.

### The Commands That Matter Most on the Job

#### `-sC` — Run Default Scripts

```bash
nmap -sC target
```

Runs the **"safe"** category of common scripts — banner grabbing, basic service info gathering. Considered standard practice in almost every real-world scan; it's low-risk and adds a lot of context for very little cost.

#### `--script vuln` — Vulnerability-Specific Scripts

```bash
nmap --script vuln target
```

Tells Nmap to run **only the scripts that hunt for known vulnerabilities** — outdated software versions, common misconfigurations, known CVEs. This is the **single most practical, result-driven step** in a typical assessment, because it directly surfaces exploitable issues instead of just service banners.

#### `--script "http-*"` — Targeted Scripts by Service (Wildcard)

```bash
nmap --script "http-*" -p80 target
```

Once you know a specific service is running (e.g. HTTP on port 80), narrow the script selection to just that service family — saves significant time versus running every script against every port.

#### The Real-World Combo Command

```bash
nmap -sC -sV --script vuln target
```

This single command — default scripts + version detection + vulnerability scripts — is what most professional pentesters run as their go-to enumeration step on a discovered host.

### What to Skip (For Now)

| Option | Why it can wait |
|---|---|
| **`--script-args`, `--script-args-file`** | Advanced script customization — not needed until you're tuning specific scripts for edge cases |
| **Boolean expressions (`and`, `or`, `not`)** | Good to understand conceptually, but rarely needed at beginner/entry level |
| **`--script-updatedb`, `--script-trace`** | Maintenance/debugging options, rarely required on the job |

---

## 💾 Output — Saving Results for Reporting

### Why This Matters

A real pentest ends with a **client-facing report** — glancing at terminal output and walking away isn't professional practice. Saving scan results in a structured, reusable format is a basic but essential habit.

### The Flags That Matter Most on the Job

#### `-oN <filename>` — Normal (Human-Readable) Output

```bash
nmap -sC -sV -oN scan_results.txt target
```

Saves output in the same readable format you'd see in the terminal — good for quick manual review or including directly in a report appendix.

#### `-oX <filename>` — XML Output

Useful when results need to be **parsed programmatically** later (e.g. feeding into another tool or a custom script).

#### `-oA <basename>` — All Formats at Once (Most Useful)

```bash
nmap -sC -sV -oA client_scan target
```

Generates **all three formats in one command**: `client_scan.nmap`, `client_scan.xml`, and `client_scan.gnmap`. This is the most efficient habit to build since you get flexibility for both human review and tool parsing without re-running the scan.

#### `--open` — Show Only Open Ports

```bash
nmap --open -p- target
```

Cuts out closed/filtered port clutter — extremely useful when scanning large networks where you only care about what's actually reachable.

### What to Skip

| Option | Why it can wait |
|---|---|
| **`-oG` (grepable), `-oS` (script kiddie)** | Deprecated/joke formats — not used in the industry today |
| **`--resume`, `--append-output`** | Niche, edge-case options not needed for standard workflows |

---

## 🛡️ Firewall / IDS Evasion

### Conceptual Overview Only

This entire category belongs to **advanced/red-team level** engagements — scenarios where an active, monitored firewall or IDS sits between you and the target, and you need to avoid detection while scanning.

At an entry level, it's enough to know these techniques **exist** and what they're for:

| Technique | Purpose (High-Level) |
|---|---|
| **`-f` (Fragmentation)** | Splits probe packets into smaller fragments to slip past simple packet-inspection rules |
| **`-D` (Decoys)** | Sends scans that appear to come from multiple fake source IPs alongside your real one, making the real attacker harder to pinpoint in logs |
| **`--spoof-mac`** | Spoofs the source MAC address to obscure the scanning machine's identity on the local network |

> 💼 **Interview tip:** If asked "what are some IDS evasion techniques in Nmap?", it's enough to answer: *"fragmentation, decoy scanning, and MAC/IP spoofing."* Actually practicing these is better saved for when you're working on Active Directory or advanced red-team engagements — they're more complex and rarely tested in beginner labs.

---

## ⚡ Quick Revision Cheat Sheet

```bash
# NSE — vulnerability & script scanning
nmap -sC target                          # Default safe scripts
nmap --script vuln target                # Vulnerability-hunting scripts
nmap --script "http-*" -p80 target       # Service-specific scripts
nmap -sC -sV --script vuln target        # The daily-driver combo command

# Output — professional result saving
nmap -sC -sV -oN scan_results.txt target # Human-readable output
nmap -sC -sV -oX scan_results.xml target # XML (for tool parsing)
nmap -sC -sV -oA client_scan target      # All formats at once (recommended)
nmap --open -p- target                   # Only show open ports

# Firewall/IDS evasion (conceptual — advanced/red-team use)
nmap -f target                           # Packet fragmentation
nmap -D RND:5 target                     # Decoy scanning
nmap --spoof-mac 0 target                # Random MAC spoofing
```

---

## 🎯 Key Takeaways

- **NSE turns Nmap from a port scanner into a lightweight vulnerability scanner** — `-sC -sV --script vuln` is the single most valuable command combo to build muscle memory around.
- **Saving output isn't optional in real engagements** — `-oA` should become an automatic habit on every scan, since it costs nothing extra and saves re-running scans later.
- **`--open` is a simple but high-value flag** for keeping large-network scans readable.
- **Firewall/IDS evasion is a "know it exists" topic at entry level** — understand the concepts (fragmentation, decoys, MAC spoofing) for interviews, but prioritize practicing NSE and output habits first.
- If only three things are memorized from this whole section, they should be: **`-sC -sV --script vuln`**, **`-oA`**, and **`--open`**.

---

<p align="center">
  <i>📌 Part of my TryHackMe learning journey — see <b>01-Nmap-Host-Discovery.md</b>, <b>02-Nmap-Basic-Port-Scanning.md</b>, and <b>03-Nmap-Advanced-Scanning.md</b> for the fundamentals this writeup builds on!</i>
</p>
