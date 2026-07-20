# 🎯 Active Reconnaissance — TryHackMe Writeup

![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen) ![Category](https://img.shields.io/badge/Category-Network%20Security-blue) ![Module](https://img.shields.io/badge/Module-Network%20Security%20%232-purple)

**Room Link:** [tryhackme.com/room/activerecon](https://tryhackme.com/room/activerecon)

---

## 📖 Overview

This room is the second stop in TryHackMe's Network Security module, picking up right where Passive Recon left off. Instead of gathering intel from public sources, this room shifts to **direct engagement** with a target — sending real traffic, making real connections, and probing real services. That directness is exactly what makes active recon more powerful *and* more detectable: every packet sent can end up in a log, an IDS alert, a WAF block, or a honeypot trigger.

> **Core Lesson:** Active recon trades stealth for depth. You learn things passive methods simply can't reveal — live services, open ports, exact software versions — but you leave footprints doing it. Never do this without explicit, signed authorization.

---

## 🎯 What This Room Covers

| # | Tool | Purpose |
|---|---|---|
| 1 | 🌐 Web Browser + DevTools | Inspect headers, JS source, cookies, and TLS certs |
| 2 | 🏓 `ping` | Check reachability + infer OS via TTL |
| 3 | 🗺️ `traceroute` / `mtr` | Map the network path and hop count to a target |
| 4 | ⌨️ `telnet` | Legacy banner grabbing over cleartext |
| 5 | 🔧 `netcat` (`nc`) | Modern banner grabbing, port probing, client/server comms |

---

## 1️⃣ Web Browser & Developer Tools 🌐

The browser is arguably the *most* useful active recon tool — it's on every machine, and its traffic is indistinguishable from a normal user browsing the web.

**Transport basics:**
- 🔌 Port 80 → plain HTTP (rare now, mostly auto-redirects to HTTPS)
- 🔒 Port 443 → HTTPS (the modern default)
- ⚡ HTTP/3 → runs over **QUIC** on UDP 443, combining TCP+TLS into one protocol; visible in DevTools as `h3` in the protocol column

You can also hit non-standard ports directly by specifying them in the URL (`https://target.com:8443/`).

**DevTools shortcut:** `Ctrl+Shift+I` (Windows/Linux) or `Option+Command+I` (macOS).

| Tab | Recon value |
|---|---|
| 📡 **Network** | Request/response headers (`Server`, `X-Powered-By`, CSP), timing, status codes, cookies |
| 💻 **Console** | Run JS snippets, inspect the live DOM |
| 📂 **Sources** | Read raw JS/CSS/HTML — frequently leaks hardcoded API endpoints, internal paths, dev comments |
| 🍪 **Application** | Cookies, LocalStorage, SessionStorage — sometimes leaking tokens or API keys client-side |
| 🔐 **Security** | Certificate issuer, validity, and **SANs** (Subject Alternative Names) — SANs often expose extra subdomains |

**Useful extensions:**
- 🦊 **FoxyProxy** — quick switching between Burp Suite/ZAP/SOCKS5 proxies
- 🎭 **User-Agent Switcher** — emulate other browsers/devices to surface version-specific behavior
- 🕵️ **Wappalyzer** — passively fingerprints CMS, frameworks, analytics, CDNs
- Alternatives: **BuiltWith**, **WhatRuns**, **Library Detector**

⚠️ Even "normal" browsing can trip behavioral WAF rules if patterns look off — rapid loads, tampered headers, frequent DevTools use, unusual User-Agents.

---

## 2️⃣ Ping — Is Anybody Home? 🏓

`ping` uses **ICMP** to send an **Echo Request (type 8)** and listen for an **Echo Reply (type 0)** — the simplest possible reachability check.

```bash
ping -c 5 MACHINE_IP        # Linux/macOS
ping -n 5 MACHINE_IP        # Windows
ping -4 -c 5 MACHINE_IP     # force IPv4
ping -6 -c 5 MACHINE_IPV6   # force IPv6
```

### 🔎 Reading TTL for OS fingerprinting

The **TTL** field isn't really about time — it's the max number of router hops a packet can survive before being dropped, decremented by one at every hop.

| Starting TTL | Typical OS |
|---|---|
| 64 | Linux |
| 128 | Windows |

A reply showing TTL 58 (not exactly 64) doesn't mean a different OS — it likely means a Linux host **six hops away**, since each hop shaves one off.

### 📋 Quick Reference

| Result | Likely meaning | Next step |
|---|---|---|
| ✅ Fast replies, low/no loss | Target online, ICMP allowed | Move to port scanning |
| ❌ "Destination Host Unreachable" | Target down / no route | Check if machine is powered on |
| 🚫 100% loss, no error | ICMP filtered/blocked | Try TCP/UDP discovery via Nmap |
| 🐢 High latency/heavy loss | Congestion, distance, or filtering | Investigate with `traceroute` |

**Reality check:** modern firewalls (incl. Windows Firewall by default), cloud providers (AWS/Azure/GCP), and WAFs/CDNs frequently block ICMP outright — a non-response doesn't always mean "target is down."

---

## 3️⃣ Traceroute — Mapping the Path 🗺️

`traceroute` (Linux/macOS) / `tracert` (Windows) reveals every router hop between you and a target by exploiting the same TTL mechanism as ping.

**How it works:** it sends packets with incrementing TTL values starting at 1. Each time a packet's TTL hits zero, that router drops it and sends back an **ICMP Time-to-Live Exceeded** message — revealing its IP. TTL=1 reveals hop 1, TTL=2 reveals hop 2, and so on until the destination itself replies.

```bash
traceroute MACHINE_IP          # default: UDP probes
traceroute -T MACHINE_IP       # TCP-based (bypasses UDP filters)
traceroute -I MACHINE_IP       # ICMP-based
traceroute -6 MACHINE_IPV6     # IPv6
mtr MACHINE_IP                 # real-time continuous view w/ stats
```

**Key realities from the room's own traceroute examples:**
- 🔀 Routes are **not fixed** — dynamic routing (BGP/OSPF), load balancing, and anycast (common with CDNs like Cloudflare/Akamai) mean two consecutive runs can return a completely different hop count and different routers entirely.
- 🚫 Some routers are deliberately configured to suppress ICMP TTL-Exceeded replies (shown as `*` in output) — a common hardening measure against recon.
- 🏁 The final hop matches the destination IP, confirming trace completion; everything before it is an intermediate router.

---

## 4️⃣ Telnet — Legacy Banner Grabbing ⌨️

**TELNET** (1969!) was built for remote CLI access, defaulting to **port 23**. It sends everything — including credentials — in **cleartext**, which is why **SSH** replaced it as the secure standard. But its raw TCP connection ability still makes it useful for one thing: **banner grabbing**.

```bash
telnet MACHINE_IP 80
GET / HTTP/1.1
host: example
[press Enter twice]
```

A web server will typically respond with headers revealing exactly what's running:

```
Server: nginx/1.6.2
```

That single header — software name + version — is exactly what you'd cross-reference against CVE databases or Exploit-DB for known vulnerabilities.

**Limitation:** telnet can't handle encrypted services. For HTTPS or TLS-wrapped ports, the room points to `curl --head https://MACHINE_IP` or `openssl s_client -connect MACHINE_IP:443` instead.

---

## 5️⃣ Netcat — The Modern Swiss Army Knife 🔧

**Netcat (`nc`)** does everything telnet does for banner grabbing, but also works as a listener/server — making it far more flexible.

### 📡 Banner grabbing (client mode)
```bash
nc MACHINE_IP 80
GET / HTTP/1.1
host: netcat
[Shift+Enter if needed]
```
Same principle as telnet — connect, send a minimal protocol request, read what comes back. FTP (port 21) and SMTP (port 25) servers often banner themselves immediately with zero input required.

### 🎧 Listening (server mode)
```bash
nc -vnlp 1234        # listen on port 1234
nc MACHINE_IP 1234   # connect from the other side
```

| Flag | Meaning |
|---|---|
| `-l` | Listen mode |
| `-p` | Specify port (must sit directly before the port number) |
| `-n` | Numeric only — skip DNS resolution |
| `-v` / `-vv` | Verbose / very verbose |
| `-k` | Keep listening after a client disconnects |
| `-6` | IPv6 listening |

📝 Ports below 1024 require root privileges to bind. For encrypted transfers, `ncat --ssl` or pairing `nc` with `stunnel` are the go-to options.

---

## 🧾 Command Cheat Sheet

| Command | Example |
|---|---|
| Ping | `ping -c 10 MACHINE_IP` (Linux/macOS) / `ping -n 10 MACHINE_IP` (Windows) |
| Ping (IPv6) | `ping -6 MACHINE_IPV6` or `ping6 MACHINE_IPV6` |
| Traceroute | `traceroute MACHINE_IP` / `tracert MACHINE_IP` (Windows) |
| Traceroute (IPv6) | `traceroute -6 MACHINE_IPV6` / `traceroute6 MACHINE_IPV6` |
| Continuous path monitor | `mtr MACHINE_IP` |
| Telnet banner grab | `telnet MACHINE_IP PORT` |
| Netcat client | `nc MACHINE_IP PORT` |
| Netcat server | `nc -lvnp PORT` |
| Netcat (IPv6) | `nc -6 MACHINE_IPV6 PORT` |
| HTTP banner via curl | `curl -I http://MACHINE_IP` / `curl -I https://MACHINE_IP` |

| OS | DevTools Shortcut |
|---|---|
| Linux / Windows | `Ctrl + Shift + I` |
| macOS | `Option + Command + I` |

---

## 🧠 Key Lessons

- 🌐 **The browser is stealthy by nature** — it blends into normal traffic while quietly leaking headers, JS source, and certificate data.
- 🏓 **TTL tells a story** — reachability *and* rough OS/hop-distance in one field.
- 🗺️ **Traceroute paths aren't stable** — CDNs, anycast, and load balancing mean don't expect the same route twice.
- ⌨️ **Banner grabbing is protocol-agnostic** — connect, read, optionally send a command; works the same whether it's telnet or netcat.
- ⚖️ **Active recon requires signed authorization** — unlike passive recon, this leaves traces and can be illegal without a scoped, signed agreement.

---

## 🛠️ Tools & Techniques Used

`Browser DevTools` `Wappalyzer` `FoxyProxy` `ping` `traceroute` `mtr` `telnet` `netcat` `curl` `Banner Grabbing` `TTL/OS Fingerprinting`

---

## ✅ Status: Completed

*Room completed independently on TryHackMe. Writeup reflects my own notes and understanding of the material — specific flag values are intentionally omitted.*
