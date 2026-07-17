# 🛰️ Nmap Host Discovery — TryHackMe Writeup

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Network%20Scanning-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)

> **Room:** [Nmap](https://tryhackme.com/room/furthernmap) — TryHackMe
> **Author of writeup:** *Your Name Here*
> **Purpose:** Understanding how Nmap discovers *live* hosts on a network before any port scanning happens — using ARP, ICMP, TCP, and UDP techniques — foundational knowledge for any network reconnaissance or penetration testing engagement.

---

## 📑 Table of Contents

- [Introduction](#-introduction)
- [The SYN Flag Fundamentals](#-the-syn-flag-fundamentals)
- [Supplying Targets: -iL vs -sL](#-supplying-targets--il-vs--sl)
- [Host Discovery Only: -sn](#-host-discovery-only--sn)
- [IP Ranges & Subnetting Concepts](#-ip-ranges--subnetting-concepts)
- [Host Discovery Techniques](#-host-discovery-techniques-protocol-by-protocol)
- [TCP Flags Reference](#-tcp-flags-reference)
- [ARP vs TCP — Different Layers](#-arp-vs-tcp--different-layers-different-jobs)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 📝 Introduction

Before scanning ports on a target, the first step in any reconnaissance workflow is figuring out **which hosts on a network are actually alive**. Nmap offers multiple host discovery techniques that rely on different protocols — **ARP, ICMP, TCP, and UDP** — each useful in different network conditions (local vs remote, firewall present vs absent).

This writeup documents my notes on every host discovery technique and flag covered while completing the **Nmap** room on TryHackMe, along with the underlying networking concepts (TCP flags, CIDR, subnetting) needed to understand *why* each scan works the way it does.

---

## 🔍 The SYN Flag Fundamentals

```bash
sudo nmap -sS example.com
```

| Step | What Happens |
|---|---|
| 1 | Nmap sends a **SYN packet** to the target port — the first step of the TCP 3-way handshake |
| 2a | If **open** → target replies **SYN/ACK** → Nmap immediately sends **RST** to tear the connection down |
| 2b | If **closed** → target replies directly with **RST** |

Because the full handshake is never completed, this is called a **half-open scan** — fast, and it leaves fewer logs on the target compared to a full connection. It requires **raw socket privileges** (`sudo`/root) to craft the packet manually.

---

## 📂 Supplying Targets: `-iL` vs `-sL`

These two flags are commonly confused but do **completely different things**.

| Flag | Category | What It Does |
|---|---|---|
| **`-iL <file>`** | Input source | Reads target IPs/hostnames **from a file** instead of the command line. A full normal scan (discovery + ports) still runs on every target. |
| **`-sL`** | Scan type | **List Scan** — sends **zero packets**. Only does reverse-DNS lookups and prints the list of IPs that *would* be scanned. A dry-run/preview only. |

```bash
nmap -iL targets.txt          # Reads targets from file, scans them normally
nmap -sL 10.10.10.0/24        # Just lists IPs in range, sends no packets
nmap -sL -iL targets.txt      # Combine: preview targets pulled from a file
```

> 💡 **In short:** `-iL` = *where* the targets come from. `-sL` = *what* to do with them (nothing — just list).

---

## 🎯 Host Discovery Only: `-sn`

```bash
sudo nmap -sn 10.200.6.0/24
```

- `-sn` (formerly `-sP`) tells Nmap: **"Only check if hosts are alive — do NOT scan any ports."**
- If `-sn` is **omitted**, Nmap discovers live hosts **and then automatically runs a full port scan** on each one found.
- Extremely useful for quick network mapping / asset discovery without the noise of a full port scan.

---

## 🌐 IP Ranges & Subnetting Concepts

### Consecutive IP Ranges

```bash
nmap 10.11.12.15-20
```
Scans **6 consecutive IPs** within the same local subnet: `.15, .16, .17, .18, .19, .20` — only the last octet varies, meaning all hosts belong to the same network segment.

### CIDR Notation (`/xx`) & Subnet Masks

```bash
nmap MACHINE_IP/30
```

CIDR notation represents the **subnet mask** — it tells Nmap how many of the 32 bits in an IPv4 address are reserved for the **network** portion vs the **host** portion.

**Formula:** `Total IPs = 2^(32 − CIDR number)`

| CIDR | Host Bits | Total IPs |
|:---:|:---:|:---:|
| /30 | 2 | 4 |
| /29 | 3 | 8 |
| /28 | 4 | 16 |
| /27 | 5 | 32 |
| /24 | 8 | 256 |
| /16 | 16 | 65,536 |

### 🏷️ Network Address vs Broadcast Address

Every subnet reserves its **first** and **last** address. Example: `192.168.1.0/30` → 4 total addresses: `.0, .1, .2, .3`

| Address | Role |
|---|---|
| `192.168.1.0` *(first)* | **Network Address** — identifies the subnet itself; not assigned to any device |
| `192.168.1.1` – `.2` | **Usable Host Addresses** — assignable to actual devices |
| `192.168.1.3` *(last)* | **Broadcast Address** — a packet sent here reaches *every* device on the subnet at once |

> 📝 Nmap still scans **all 4 addresses** in a `/30` (including network & broadcast), even though only 2 are technically "usable" hosts.

---

## 🖧 Host Discovery Techniques (Protocol by Protocol)

| Scan Type | Example Command | How It Works |
|---|---|---|
| **ARP Scan** | `sudo nmap -PR -sn 10.200.6.0/24` | Sends a Layer 2 broadcast ARP request ("Who has this IP?"). Only works on the **local network/broadcast domain**. Fast & the default best method for LAN discovery. |
| **ICMP Echo Scan** | `sudo nmap -PE -sn 10.200.6.0/24` | Exactly what a normal `ping` does — sends an **ICMP Echo Request**, expects an **ICMP Echo Reply**. Any reply = host alive. |
| **ICMP Timestamp Scan** | `sudo nmap -PP -sn 10.200.6.0/24` | Fallback when Echo (ping) is blocked. Asks the target for its current system time — any reply confirms the host is alive (the time value itself doesn't matter). |
| **ICMP Address Mask Scan** | `sudo nmap -PM -sn 10.200.6.0/24` | Another Echo fallback — asks the target for its **subnet mask**. A reply confirms it's up. Mostly legacy — modern OSes rarely respond. |
| **TCP SYN Ping Scan** | `sudo nmap -PS22,80,443 -sn 10.200.6.0/30` | Sends a SYN packet to specific ports for **discovery only**. Either SYN/ACK (open) or RST (closed) confirms the host is alive. Useful when ICMP is blocked but common ports (80, 443, 22) aren't. |
| **TCP ACK Ping Scan** | `sudo nmap -PA22,80,443 -sn 10.200.6.0/30` | Sends an ACK as if a connection already exists. The confused target replies RST — confirming it's alive. Bypasses **stateless firewalls** that block SYN but allow ACK. |
| **UDP Ping Scan** | `sudo nmap -PU53,161,162 -sn 10.200.6.0/30` | Sends a UDP datagram to specific ports (DNS-53, SNMP-161/162). A closed port usually triggers an **ICMP "Port Unreachable"** error, confirming the host is alive. |

> ⚠️ **Key principle:** Any response from a host — regardless of protocol — indicates that it is online. That's the entire concept behind host discovery.

---

## 🚩 TCP Flags Reference

Understanding TCP flags is essential to understanding *why* each scan type behaves the way it does:

| Flag | Function | Analogy |
|---|---|---|
| **SYN** | Initiates a new connection | The first "hello" of a handshake |
| **ACK** | Confirms data/packet receipt | WhatsApp's "seen" tick |
| **PSH** | Push data to the application immediately, don't buffer | "Send it now, don't wait" |
| **RST** | Abruptly tear down the connection | Knocking on a closed door — no answer |
| **FIN** | Gracefully/politely close the connection | "Alright, hanging up now" |
| **URG** | Marks data as urgent, process immediately | An "URGENT" stamped letter |

### The TCP 3-Way Handshake (Connection Establishment)
```
Client → Server :  SYN
Server → Client :  SYN + ACK
Client → Server :  ACK
```

### The TCP 4-Way Close (Graceful Termination)
```
Client → Server :  FIN
Server → Client :  ACK
Server → Client :  FIN
Client → Server :  ACK
```

---

## 🔗 ARP vs TCP — Different Layers, Different Jobs

A common point of confusion: does ARP let you *communicate* with a host, the way a TCP handshake does?

| | ARP | TCP Handshake |
|---|---|---|
| **OSI Layer** | Layer 2 (Data Link) | Layer 4 (Transport) |
| **Purpose** | Resolve IP → MAC address mapping | Establish a real, usable session for data exchange |
| **Data Transfer?** | ❌ No — just a lookup | ✅ Yes — full application data can flow |

**Relationship:** Whenever a device sends a TCP packet to another host on the same local network, the OS first performs ARP in the background (if the MAC isn't cached) to resolve the destination MAC address — then wraps the TCP packet in an Ethernet frame addressed to that MAC.

> 🏠 **Analogy:** ARP is like finding out someone's house/flat number in a housing society. The TCP handshake is like actually knocking on their door and having a conversation. You need the address first, but knowing it isn't the same as communicating.

---

## ⚡ Quick Revision Cheat Sheet

```
DISCOVERY FLAG             →  PROTOCOL USED           →  FALLBACK FOR
──────────────────────────────────────────────────────────────────────
-PR                        →  ARP                     →  (default on LAN)
-PE                        →  ICMP Echo                →  (default ping)
-PP                        →  ICMP Timestamp           →  -PE blocked
-PM                        →  ICMP Address Mask        →  -PE blocked
-PS<ports>                 →  TCP SYN                  →  ICMP blocked
-PA<ports>                 →  TCP ACK                  →  SYN/ICMP blocked
-PU<ports>                 →  UDP                      →  All TCP/ICMP blocked

TARGET INPUT               →  BEHAVIOUR
──────────────────────────────────────────────────────────────────────
-iL targets.txt             →  Read targets from file, scan normally
-sL <targets>               →  Preview only, zero packets sent

SCAN SCOPE                 →  BEHAVIOUR
──────────────────────────────────────────────────────────────────────
-sn                         →  Discovery only, no port scan
(no -sn)                    →  Discovery + full port scan
```

---

## 🎯 Key Takeaways

- **`-sS`** is the foundation of Nmap's stealthy scanning philosophy — half-open connections mean fewer logs.
- **`-iL`** controls *where targets come from*; **`-sL`** controls *whether any packets are sent at all* — never confuse the two.
- **`-sn`** is the single most important flag for pure host discovery — pairing it with the right `-P*` flag lets you adapt to firewalled environments.
- CIDR notation and the network/broadcast address concept are essential for correctly scoping a scan to an entire subnet without wasting time on invalid targets.
- ARP and TCP operate on entirely different OSI layers — ARP only *locates* a host, it never *communicates* with it the way an actual TCP session does.

---

<p align="center">
  <i>📌 Part of my TryHackMe learning journey — see <b>02-Nmap-Basic-Port-Scanning.md</b> next for TCP/UDP port scanning, service/version detection, and OS fingerprinting!</i>
</p>
