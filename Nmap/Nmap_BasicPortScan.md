# 🔎 Nmap Basic Port Scanning — TryHackMe Writeup

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Network%20Scanning-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)

> **Room:** [Nmap](https://tryhackme.com/room/furthernmap) — TryHackMe
> **Author of writeup:** *Your Name Here*
> **Purpose:** Understanding the core Nmap port scanning techniques (TCP Connect, SYN, UDP) plus service/version detection and OS fingerprinting — essential skills for identifying attack surface during a penetration test.

---

## 📑 Table of Contents

- [Introduction](#-introduction)
- [Core Port Scan Types](#-core-port-scan-types)
- [Service, Version & OS Detection](#-service-version--os-detection)
- [Specifying Targets for Port Scans](#-specifying-targets-for-port-scans)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 📝 Introduction

Once live hosts are discovered on a network (see [`01-Nmap-Host-Discovery.md`](./01-Nmap-Host-Discovery.md)), the next step is figuring out **which ports are open** on those hosts and **what services** are running on them. Nmap supports several port scanning techniques, each with different trade-offs in speed, stealth, and required privileges.

This writeup documents my notes on port scanning while completing the **Nmap** room on TryHackMe.

---

## 🧩 Core Port Scan Types

| Scan Type | Example Command | Privileges Needed |
|---|---|---|
| **TCP Connect Scan** | `nmap -sT 10.112.189.144` | None (regular user) |
| **TCP SYN Scan** | `sudo nmap -sS 10.112.189.144` | Root/Admin (raw sockets) |
| **UDP Scan** | `sudo nmap -sU 10.112.189.144` | Root/Admin |

### 🔹 `-sT` — TCP Connect Scan

Completes the **full 3-way TCP handshake**:

```
Client → Server :  SYN
Server → Client :  SYN/ACK
Client → Server :  ACK        ← full connection is established here
```

- Uses the OS's normal socket API — no special privileges required.
- Because the connection is fully established, it's **noisier** — generates more logs, including at the application level on the target.
- Nmap automatically falls back to `-sT` when it doesn't have root privileges to perform `-sS`.

### 🔹 `-sS` — TCP SYN Scan ("Half-Open" Scan)

- Sends only a **SYN** packet.
- **Open port** → target replies **SYN/ACK** → Nmap sends **RST** to tear it down without completing the handshake.
- **Closed port** → target replies **RST** directly.
- Never completes the full handshake → **faster and stealthier**, fewer logs generated.
- Requires raw socket access (`sudo`).

### 🔹 `-sU` — UDP Scan

UDP is a **connectionless protocol** — there is no handshake concept at all (unlike TCP). Nmap sends a UDP packet to the target port and interprets the response:

| Response | Port State |
|---|---|
| No response (timeout) | `open\|filtered` — ambiguous, could be open or silently dropped by firewall |
| ICMP "Port Unreachable" | **Closed** |
| Application-specific UDP reply | **Open** (uncommon — depends on the service) |

> ⚠️ Because "no response" is ambiguous, UDP scanning is considered **slower and less reliable** than TCP scanning.

---

## 🧪 Service, Version & OS Detection

```bash
sudo nmap -sS -sV -sC -O scanme.nmap.org
```

| Flag | Purpose |
|---|---|
| **`-sS`** | SYN scan — discover open ports |
| **`-sV`** | Version detection — identify the software/version running on each open port (e.g. Apache 2.4.29) |
| **`-sC`** | Runs default Nmap Scripting Engine (NSE) scripts against discovered services for extra enumeration |
| **`-O`** | OS detection — attempts to fingerprint the target's operating system |

```bash
sudo nmap -A scanme.nmap.org
```

- **`-A`** is an all-in-one aggressive scan: combines **OS detection, version detection, script scanning, and traceroute** — gives a comprehensive quick profile of a single device.

---

## 🎯 Specifying Targets for Port Scans

```bash
nmap MACHINE_IP scanme.nmap.org example.com
```
Multiple targets (IP or hostname) can be scanned in a single command.

```bash
nmap 10.11.12.15-20
```
Scans a range of 6 consecutive IPs in the same subnet: `.15` through `.20`.

```bash
nmap MACHINE_IP/30
```
CIDR notation — `/30` covers **4 total IPs** in that subnet *(see subnetting breakdown in the Host Discovery writeup for the full explanation of network/broadcast addresses)*.

```bash
nmap -iL targets.txt
```
Reads targets from a file — useful for scanning large lists (e.g. 500 IPs) in one go.

```bash
nmap -sL targetipaddress
```
**List Scan** — previews target IPs without sending any actual probe packets. Good for double-checking a target list before committing to a real scan.

```bash
nmap -PR -sn 10.200.6.0/24
```
Combines **ARP-based host discovery** with `-sn` (no port scan) — confirms which hosts are alive on the local subnet without probing any ports.

---

## ⚡ Quick Revision Cheat Sheet

```
BASIC PORT SCANS            →  DETAIL
──────────────────────────────────────────────────────────────
-sT                          →  Full handshake, no root, noisier
-sS                          →  Half-open, root needed, stealthy
-sU                          →  Connectionless, ambiguous, slower

ENRICHMENT FLAGS             →  ADDS
──────────────────────────────────────────────────────────────
-sV                          →  Service/version detection
-sC                          →  Default NSE scripts
-O                           →  OS fingerprinting
-A                           →  All of the above + traceroute

TARGET SPECIFICATION         →  BEHAVIOUR
──────────────────────────────────────────────────────────────
IP1 IP2 hostname             →  Multiple targets in one command
IP-range (15-20)             →  Consecutive IPs, same subnet
IP/CIDR                       →  Full subnet range
-iL targets.txt              →  Targets read from a file
-sL                           →  Preview only, no packets sent
```

```bash
# Basic port scans
nmap -sT 10.112.189.144                     # TCP Connect scan (no root needed)
sudo nmap -sS 10.112.189.144                # TCP SYN scan (stealthy, root needed)
sudo nmap -sU 10.112.189.144                # UDP scan

# Service/version/OS detection
sudo nmap -sS -sV -sC -O scanme.nmap.org    # Full detailed scan
sudo nmap -A scanme.nmap.org                # Aggressive all-in-one scan

# Multiple / range targets
nmap MACHINE_IP scanme.nmap.org example.com # Multiple named targets
nmap 10.11.12.15-20                         # IP range within same subnet
nmap MACHINE_IP/30                          # CIDR subnet scan (4 IPs)

# Target list handling
nmap -iL targets.txt                        # Scan targets from file
nmap -sL targetipaddress                    # Preview-only, no packets sent

# Discovery-only combined with subnet scan
nmap -PR -sn 10.200.6.0/24                  # ARP discovery, no port scan
```

---

## 🎯 Key Takeaways

- **`-sT`** = full handshake, no special privileges, noisier.
- **`-sS`** = half-open handshake, requires root, stealthier — the industry-standard default scan.
- **`-sU`** = connectionless, ambiguous results, slower — but essential since many services (DNS, SNMP, DHCP) run over UDP.
- **`-sV`, `-sC`, `-O`, `-A`** layer on top of a base scan to enrich results with service versions, script output, and OS fingerprinting.
- Combining host discovery flags (`-PR`, `-PE`, etc.) with `-sn` lets you map a network **before** committing to a full, potentially noisy port scan.

---

<p align="center">
  <i>📌 Part of my TryHackMe learning journey — see <b>01-Nmap-Host-Discovery.md</b> for ARP/ICMP/TCP/UDP host discovery, TCP flags, and subnetting fundamentals!</i>
</p>
