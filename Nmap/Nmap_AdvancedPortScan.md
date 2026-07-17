# ⚡ Advanced Nmap Scanning — TryHackMe Writeup

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Network%20Scanning-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)

> **Rooms:** Advanced Nmap / Nmap Post Port Scans (Premium) — TryHackMe *(notes based on official Nmap documentation)*
> **Author of writeup:** *Your Name Here*
> **Purpose:** Documenting the advanced port scanning techniques, port specification tricks, service/version detection, OS fingerprinting, and timing/performance controls needed for efficient, real-world Nmap usage in penetration testing engagements.

---

## 📑 Table of Contents

- [Introduction](#-introduction)
- [Advanced Port Scan Types](#-advanced-port-scan-types)
- [Port Specification & Scan Order](#-port-specification--scan-order)
- [Service and Version Detection](#-service-and-version-detection)
- [OS Detection](#-os-detection)
- [Timing and Performance](#-timing-and-performance)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 📝 Introduction

Beyond the basic `-sS` / `-sT` / `-sU` scans (covered in my earlier writeup, [`02-Nmap-Basic-Port-Scanning.md`](./02-Nmap-Basic-Port-Scanning.md)), Nmap offers a much deeper toolkit: alternative scan types that exploit protocol-level loopholes, fine-grained control over which ports get scanned, accurate service/version fingerprinting, OS detection, and — critically — timing controls that let a scan be tuned for either stealth or speed.

This writeup focuses only on the concepts and flags that are **still relevant to modern pentesting and come up in technical interviews** — legacy or largely obsolete techniques (SCTP scans, Window scan, Maimon scan, FTP bounce scan, custom `--scanflags`) have been intentionally left out to keep this practical and job-focused.

---

## 🎯 Advanced Port Scan Types

### `-sA` — TCP ACK Scan

Unlike other scans, this one **never determines whether a port is open** — its entire purpose is to **map firewall rulesets**.

```bash
nmap -sA target
```

- Sends a packet with **only the ACK flag** set (no real connection exists)
- If the system is **unfiltered** → both open and closed ports send back **RST** → Nmap labels them `unfiltered` (reachable, but open/closed state unknown)
- No response, or an ICMP error → port is `filtered`

> 💼 **Job relevance:** Core to understanding whether a target sits behind a **stateful vs stateless firewall** — a common real-world recon step.

---

### `-sN` / `-sF` / `-sX` — NULL, FIN, and Xmas Scans

These exploit a loophole in the **TCP RFC (793)**: a packet without SYN, RST, or ACK set should get an RST back if the port is closed, and **no response** if the port is open.

| Scan | Flags Set | Analogy |
|---|---|---|
| **Null (`-sN`)** | None | A completely blank packet |
| **FIN (`-sF`)** | FIN only | "I'm closing this connection" — sent without ever opening one |
| **Xmas (`-sX`)** | FIN + PSH + URG | All lights on — like a lit-up Christmas tree |

**Interpreting the response:**
- **RST received** → port `closed`
- **No response** → port `open|filtered`
- **ICMP unreachable error** → port `filtered`

✅ **Advantage:** Can sneak past certain non-stateful firewalls/packet filters, and are even stealthier than SYN scans.
❌ **Limitation:** Systems that don't strictly follow RFC 793 (Windows, many Cisco devices, IBM OS/400) send RST regardless of port state — making all ports appear `closed`. These scans are mostly reliable against **Unix/Linux-based systems**.

> 💼 **Job relevance:** Classic firewall/IDS evasion concept — commonly tested in OSCP/CEH-style interviews.

---

### `-sI <zombie host>[:<probeport>]` — Idle Scan

The most advanced scan technique here — allows a **completely blind TCP port scan** where no packets from your real IP ever touch the target.

**How it works (simplified):**
1. Find an idle "zombie" machine with **predictable IP ID sequence generation**
2. Note the zombie's current IP ID
3. Send a spoofed packet to the target, pretending to be the zombie
4. If the target port is **open** → it replies to the zombie → zombie's IP ID sequence jumps
5. If **closed** → zombie's IP ID stays the same
6. Check the zombie's IP ID again (this step is done from your real IP) to infer the target's port state

```bash
nmap -sI zombie_host:80 target
```

> 💼 **Job relevance:** Demonstrates deep understanding of IP spoofing, fragmentation IDs, and stealth scanning — great for showing depth in senior-level interviews, even if rarely used day-to-day. Also useful for mapping IP-based trust relationships between machines.

---

### `-sO` — IP Protocol Scan

Not technically a *port* scan — it determines which **IP protocols** (TCP, ICMP, IGMP, SCTP, etc.) a target supports, cycling through protocol numbers (0–255) instead of port numbers.

```bash
nmap -sO target
```

- Any response → protocol `open`
- ICMP protocol unreachable (type 3, code 2) → `closed`
- Other ICMP unreachable errors → `filtered`
- No response → `open|filtered`

> 💼 **Job relevance:** Useful for network-level enumeration; shows understanding that scanning isn't limited to just TCP/UDP ports.

---

## 📂 Port Specification & Scan Order

### `-p` — Only Scan Specified Ports

By default, Nmap scans only the **top 1000 most common ports** per protocol. `-p` overrides this.

```bash
nmap -p 80 target              # Single port
nmap -p 1-1023 target          # Range
nmap -p- target                # All 65535 ports
nmap -p U:53,111,137,T:21-25,80,139,8080 target   # Mixed UDP + TCP ports
nmap -p ftp,http* target       # Named ports with wildcards
```

- `T:` and `U:` prefixes specify TCP vs UDP ports in a combined scan (requires both `-sU` and a TCP scan type like `-sS`)
- `-p-` is the standard "scan everything" flag used before a deep dive

> 💼 **Job relevance:** The most frequently used port flag in real assessments — typically `-p-` for full discovery, followed by `-sV -sC` on the interesting ports found.

---

### `-F` — Fast (Limited Port) Scan

```bash
nmap -F target
```

Scans only the **top 100** most common ports instead of the default 1000. Useful for a quick overview when time is tight.

> 💼 **Job relevance:** Shows understanding of the speed vs thoroughness trade-off — quick recon vs full assessment.

---

### `--top-ports <n>` — Scan the N Most Common Ports

```bash
nmap --top-ports 50 target
```

More flexible than `-F` — you choose exactly how many of the most statistically common ports (based on Nmap's `nmap-services` frequency database) to scan.

> 💼 **Job relevance:** Preferred over `-F` in practice for quick recon in CTFs and time-boxed engagements, since the port count is fully customizable.

---

### `--exclude-ports <port ranges>` — Never Scan These Ports

```bash
nmap --exclude-ports 22,3389 target
```

Permanently excludes specified ports from **every** scan type, including the discovery phase.

> 💼 **Job relevance:** Used for safe/controlled scanning — e.g. when a client explicitly says not to touch certain critical ports (production SSH, RDP, honeypots).

---

### `-r` — Don't Randomize Ports

```bash
nmap -r target
```

By default, Nmap **randomizes** the scan order of ports (to make the scan pattern less predictable to an IDS). `-r` forces sequential (lowest → highest) scanning instead.

> 💼 **Job relevance:** Small but relevant stealth concept — randomization exists specifically to make port-scan patterns harder to fingerprint.

---

## 🔬 Service and Version Detection

### The Core Problem: Port Number ≠ Guaranteed Service

Nmap's `nmap-services` database (~2,200 known services) lets it **guess** a service from a port number (e.g. port 80 = HTTP). But this is only a guess — any service can run on any port. Real vulnerability assessment needs the **exact version**, not just a guess.

### `-sV` — Version Detection

```bash
nmap -sV target
```

Probes already-discovered open ports to determine:
- Service protocol (FTP, SSH, HTTP, etc.)
- Application name (Apache, OpenSSH, ISC BIND, etc.)
- **Exact version number**
- Device type & OS family
- SSL-wrapped service details (if compiled with OpenSSL support)

> 💼 **Job relevance:** The foundation of **CVE-based vulnerability hunting**. Without an exact version (e.g. "Apache 2.4.29"), you can't meaningfully search exploit databases. Mandatory in any professional pentest report.

`-A` also enables version detection, along with OS detection, scripts, and traceroute.

---

### `--version-intensity <0-9>` — Control Probe Depth

```bash
nmap -sV --version-intensity 9 target
```

- Each probe has a "rarity value" (1–9); lower numbers catch common services quickly, higher numbers catch rare ones but slow the scan down
- **Default = 7**

| Shortcut | Equivalent | Use Case |
|---|---|---|
| `--version-light` | `--version-intensity 2` | Fast recon, slightly less accurate |
| `--version-all` | `--version-intensity 9` | Thorough, exhaustive final scan |

> 💼 **Job relevance:** Real assessments are time-constrained — `--version-light` for quick recon, `--version-all` for the final, thorough report.

---

### `--allports` — Don't Skip Port 9100

```bash
nmap -sV --allports target
```

By default, Nmap **skips TCP port 9100** during version detection because some printers print anything sent to it, generating garbage output. `--allports` removes this exclusion.

> 💼 **Job relevance:** A well-known real-world gotcha — good to know when scanning printers/IoT devices to avoid unexpected side effects (and to explain unusual scan behavior).

---

## 🖥️ OS Detection

### `-O` — Enable OS Detection

```bash
nmap -O target
```

Sends a series of TCP/UDP packets and compares the responses against a database of **2,600+ known OS fingerprints**, returning vendor name, OS, generation, and device type.

> 💼 **Job relevance:** Knowing the target OS early shapes the entire exploitation strategy — Windows vs Linux vs network device all imply very different attack paths.

`-A` also enables OS detection alongside version detection and scripts.

---

### `--osscan-limit` — Skip Unpromising Targets

```bash
nmap -O --osscan-limit target
```

OS detection is only reliable when **at least one open and one closed port** are found. This flag skips OS detection entirely on hosts that don't meet that criteria — saving significant time on large-scale (`-Pn`) scans.

> 💼 **Job relevance:** Demonstrates efficient large-scale scanning strategy.

---

### `--osscan-guess` / `--fuzzy` — Guess More Aggressively

```bash
nmap -O --osscan-guess target
```

When no perfect match is found, this makes Nmap offer **near-matches** with a confidence percentage, instead of reporting nothing.

> 💼 **Job relevance:** Useful in the real world where perfect fingerprint matches are rare (custom or modified OS builds).

---

## ⏱️ Timing and Performance

### `-T <template>` — Timing Templates (MOST IMPORTANT FLAG)

```bash
nmap -T4 target
```

Instead of manually tuning many low-level timing options, Nmap offers 6 ready-made templates:

| Template | # | Behavior |
|---|---|---|
| **paranoid** | T0 | Serialized, one port at a time, 5-min wait between probes — max IDS evasion |
| **sneaky** | T1 | 15-second wait between probes |
| **polite** | T2 | Minimal bandwidth/target resource usage |
| **normal** | T3 | **Default** — no changes |
| **aggressive** | T4 | Assumes a fast, reliable network; caps scan delay at 10ms |
| **insane** | T5 | Assumes an extremely fast network; trades some accuracy for speed |

> 💼 **Industry-standard practice:** **`-T4`** is used almost universally on modern broadband/Ethernet connections — the best balance of speed and reliability. T0/T1 are reserved for genuine stealth/red-team engagements where time isn't a constraint.

---

### `--min-rate` / `--max-rate` — Directly Control Packet Rate

```bash
nmap --min-rate 300 target
nmap --max-rate 100 target
```

- `--min-rate 300` → Nmap tries to send **at least** 300 packets/sec
- `--max-rate 100` → Nmap **never exceeds** 100 packets/sec

⚠️ Scanning faster than the network can support can *reduce* accuracy — Nmap's adaptive retransmission may detect the resulting congestion and actually send **more** packets overall, making the scan slower, not faster.

> 💼 **Job relevance:** Used in enterprise-scale scanning where deadlines or bandwidth constraints must be respected precisely.

---

### `--max-retries <numtries>` — Limit Retransmissions

```bash
nmap --max-retries 2 target
```

Default is 10 retries when a probe gets no response. Lowering this speeds up scans significantly at the cost of occasionally missing ports on slow/rate-limited hosts.

> 💼 **Job relevance:** Commonly combined with `-T4`/`-T5` for fast scans in time-boxed engagements.

---

### `--host-timeout <time>` — Give Up on Slow Hosts

```bash
nmap --host-timeout 30m target
```

Some hosts are simply slow to scan (poor hardware, rate limiting, restrictive firewalls) and can eat up a majority of total scan time. This sets a hard ceiling — a host exceeding it is **skipped entirely** (no port table, OS, or version results for it).

> 💼 **Job relevance:** Essential when scanning an entire subnet — prevents a handful of problematic hosts from stalling the whole engagement.

---

### `--scan-delay` / `--max-scan-delay` — Adjust Delay Between Probes

```bash
nmap --scan-delay 1s target
```

Forces a minimum wait time between probes sent to a given host. Two main uses:

1. **Matching rate-limited hosts** (e.g. Solaris systems that only respond to one ICMP message per second)
2. **Evading threshold-based IDS/IPS** (e.g. Snort) — spacing probes out to stay under detection thresholds

> 💼 **Job relevance:** Core IDS evasion concept, relevant in red-team/stealth engagement discussions.

---

### `--defeat-rst-ratelimit` / `--defeat-icmp-ratelimit` — Trade Accuracy for Speed

```bash
nmap --defeat-rst-ratelimit target
```

Some systems rate-limit their RST or ICMP error responses, which slows Nmap down as it adapts its timing. These flags tell Nmap to **ignore** those rate limits and move faster — at the cost of some ports being mislabeled (`filtered` instead of `closed`, or `closed|filtered` instead of `open|filtered`).

> 💼 **Job relevance:** Advanced trade-off discussion — useful when only "open" ports matter and the closed/filtered distinction isn't worth the extra time.

---

## ⚡ Quick Revision Cheat Sheet

```bash
# Advanced port scans
nmap -sA target                          # ACK scan — maps firewall rules
nmap -sN target                          # Null scan — no flags set
nmap -sF target                          # FIN scan
nmap -sX target                          # Xmas scan — FIN+PSH+URG
nmap -sI zombie_host:80 target           # Idle scan — blind, spoofed scan
nmap -sO target                          # IP protocol scan

# Port specification
nmap -p 80 target                        # Single port
nmap -p- target                          # All 65535 ports
nmap -p U:53,T:21-25,80 target           # Mixed UDP/TCP ports
nmap -F target                           # Fast scan (top 100 ports)
nmap --top-ports 50 target               # Top N common ports
nmap --exclude-ports 22,3389 target      # Never scan these ports
nmap -r target                           # Sequential (no randomization)

# Service/version detection
nmap -sV target                          # Version detection
nmap -sV --version-intensity 9 target    # Max probe intensity
nmap -sV --version-light target          # Fast, less accurate
nmap -sV --version-all target            # Exhaustive, most accurate
nmap -sV --allports target               # Don't skip port 9100

# OS detection
nmap -O target                           # OS fingerprinting
nmap -O --osscan-limit target            # Skip unpromising hosts
nmap -O --osscan-guess target            # More aggressive guessing

# Timing and performance
nmap -T4 target                          # Industry-standard timing template
nmap --min-rate 300 target               # Minimum packet rate
nmap --max-rate 100 target                # Maximum packet rate
nmap --max-retries 2 target               # Limit retransmissions
nmap --host-timeout 30m target            # Skip slow hosts
nmap --scan-delay 1s target               # Delay between probes (IDS evasion)
```

---

## 🎯 Key Takeaways

- **`-sA`** never reveals port state — it's purely a **firewall-mapping** tool.
- **`-sN` / `-sF` / `-sX`** exploit a TCP RFC loophole for stealth, but are unreliable against Windows/Cisco systems that don't follow RFC 793 strictly.
- **`-sI` (Idle scan)** is the ultimate blind-scanning technique — conceptually important even if rarely used day-to-day.
- **`-p-`** and **`--top-ports`** are the two most practical port-specification flags for real engagements.
- **`-sV`** is non-negotiable for real vulnerability assessment — exact versions, not just port guesses, are what let you match CVEs.
- **`-O`** shapes exploitation strategy early by revealing the target OS.
- **`-T4`** is the de facto industry-standard timing template; fine-grained options (`--min-rate`, `--max-retries`, `--host-timeout`, `--scan-delay`) exist for precise control when a deadline, bandwidth limit, or IDS evasion requirement demands it.

---

<p align="center">
  <i>📌 Part of my TryHackMe learning journey — see <b>01-Nmap-Host-Discovery.md</b> and <b>02-Nmap-Basic-Port-Scanning.md</b> for the fundamentals this writeup builds on!</i>
</p>
