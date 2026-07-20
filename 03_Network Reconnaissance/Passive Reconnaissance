# 🕶️ Passive Reconnaissance — TryHackMe Writeup

![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen) ![Category](https://img.shields.io/badge/Category-Network%20Security-blue) ![Module](https://img.shields.io/badge/Module-Network%20Security%20%231-purple)

**Room Link:** [tryhackme.com/room/passiverecon](https://tryhackme.com/room/passiverecon)

---

## 📖 Overview

This room is the first stop in TryHackMe's Network Security module and lays the theoretical + tool-based foundation for everything that follows (Active Recon, Nmap, Protocols & Servers). It covers **passive reconnaissance** — gathering intelligence about a target using only publicly available sources, without ever sending a single packet directly to it.

> **Core Lesson:** You can learn an enormous amount about a target — who owns it, what servers it runs, what subdomains exist, what's exposed to the internet — without the target ever knowing you looked. That's the entire point of "passive."

---

## 🎯 What This Room Covers

| # | Topic | Tool/Service | Purpose |
|---|---|---|---|
| 1 | Passive vs Active Recon | Theory | Understand the distinction and legal implications |
| 2 | WHOIS Lookups | `whois` | Domain registration, registrar, and ownership data |
| 3 | DNS Lookups | `nslookup` / `dig` | Resolve IPs, mail servers, TXT records |
| 4 | Subdomain Discovery | DNSDumpster | Find subdomains a normal DNS query can't reveal |
| 5 | Internet-Wide Device Search | Shodan.io | Discover exposed services/devices without connecting |

---

## 1️⃣ Passive vs. Active Recon — The Core Distinction 🎭

The room frames reconnaissance as the very first step of any attack (or defensive audit) — mirrored in models like the Unified Kill Chain. It splits recon into two flavors:

- 🕶️ **Passive Recon** — relying entirely on publicly available information. Think: reading a company's job postings, browsing their Facebook page, or querying a public DNS server. No direct engagement with the target's systems at all.
- 🎯 **Active Recon** — direct engagement with the target: connecting to their web/FTP/mail servers, calling employees under a pretext (social engineering), or physically visiting a location.

**Why the distinction matters legally:** active recon can quickly cross into unauthorized access territory without a signed engagement scope, since it directly touches the target's infrastructure. Passive recon, by contrast, uses only information that's already public.

**Examples given in the room (conceptually):**
- 📘 Browsing a target's public social media for employee names → **Passive**
- 📡 Pinging a company's web server to check if ICMP is blocked → **Active**
- 🍻 Chatting up an IT admin at a party to extract infrastructure details → **Active** (social engineering counts as direct engagement, even if informal)

---

## 2️⃣ WHOIS — Domain Ownership Records 📋

**Protocol:** WHOIS runs over **TCP port 43**, following the RFC 3912 spec. Domain registrars maintain these records for every domain they lease out.

**What WHOIS reveals:**

| Field | Info |
|---|---|
| 🏢 Registrar | Which company registered the domain |
| 👤 Registrant Contact | Name, org, address, phone (unless privacy-shielded) |
| 📅 Dates | Creation, last update, expiration |
| 🌐 Name Servers | Which DNS servers to query for this domain |

**Command:**
```bash
whois DOMAIN_NAME
```

Running this against a target domain typically shows which registrar is managing it, when it was first registered and when it expires, and the authoritative name servers. Many registrants shield their personal contact info behind a privacy service specifically because automated tools have historically scraped WHOIS records to harvest email addresses for spam — so don't expect registrant PII to always be visible.

**Why it matters for a pentest:** the collected data can reveal new angles of attack — e.g., targeting the admin's email server, or investigating the DNS provider — assuming those fall within engagement scope.

---

## 3️⃣ nslookup & dig — Querying DNS 🔎

While WHOIS tells you *who owns* a domain, DNS lookups tell you *what infrastructure* it points to.

### `nslookup`
```bash
nslookup -type=TYPE DOMAIN_NAME SERVER
```

| Parameter | Meaning |
|---|---|
| `TYPE` | Record type to query (see table below) |
| `DOMAIN_NAME` | Target domain |
| `SERVER` | Optional — which DNS resolver to query (e.g., `1.1.1.1`, `8.8.8.8`, `9.9.9.9`) |

**Common DNS record types:**

| Query type | Result |
|---|---|
| `A` | IPv4 address(es) |
| `AAAA` | IPv6 address(es) |
| `CNAME` | Canonical name (alias) |
| `MX` | Mail exchange servers |
| `SOA` | Start of Authority |
| `TXT` | Free-text records (often SPF, verification tokens, etc.) |

Example: `nslookup -type=A tryhackme.com 1.1.1.1` returns every IPv4 address associated with the domain — each of which becomes a candidate for further scanning within scope. Similarly, `-type=MX` reveals which mail provider handles a domain's email (e.g., discovering a target routes mail through Google Workspace tells you the mail servers themselves are unlikely to be a soft target, since Google manages patching).

### `dig` — "Domain Information Groper"
```bash
dig DOMAIN_NAME TYPE
dig @SERVER DOMAIN_NAME TYPE
```

`dig` is the more detailed, modern alternative — by default it surfaces extra data like **TTL (Time To Live)** values for each record, which `nslookup` doesn't show as prominently. Both tools query the same public DNS infrastructure, so neither one alerts the target.

---

## 4️⃣ DNSDumpster — Automated Subdomain Discovery 🌐

Plain `nslookup`/`dig` queries can't discover subdomains on their own — you'd need to already know a subdomain exists before you could query it. That's where **[DNSDumpster](https://dnsdumpster.com/)** comes in.

**What it does:** aggregates DNS reconnaissance into one query — combining search-engine-style discovery and known-subdomain databases to reveal subdomains that a direct query would never surface (e.g., an old, rarely-updated `wiki.` or `webmail.` subdomain that isn't linked anywhere public).

**What you get back:**
- 📜 A table of DNS servers, resolved IPs, and rough geolocation
- 📧 MX records resolved to IPs with ownership/location info
- 📝 TXT records
- 🕸️ A visual graph connecting DNS/MX branches to their respective servers, which can be exported and rearranged

**Why forgotten subdomains matter:** a subdomain that isn't part of the regular update/patch cycle is exactly the kind of soft target that ends up running outdated, vulnerable software — all discoverable without touching the target directly.

---

## 5️⃣ Shodan.io — The Search Engine for Devices 🛰️

**[Shodan.io](https://www.shodan.io/)** flips the concept of a search engine: instead of indexing web pages, it continuously connects to devices reachable on the internet, and indexes *them* — routers, IoT devices, industrial control systems, and of course web servers.

**Information typically surfaced per result:**
- 🌍 IP address & geographic location
- 🏢 Hosting provider
- 🖥️ Server software type and version

**Why it's "passive":** you're querying Shodan's own database of previously-collected scan data — you never connect to the target device yourself. This makes it a favorite for both:
- 🔴 **Red teamers** — scoping a target's exposed footprint before an engagement
- 🔵 **Blue teamers** — auditing what their own organization is inadvertently exposing to the internet

You can search by domain, IP, organization name, or specific service banners, and Shodan's own help documentation covers the full query syntax for advanced searches.

---

## 🧾 Command Cheat Sheet

| Purpose | Command |
|---|---|
| WHOIS lookup | `whois tryhackme.com` |
| DNS A record | `nslookup -type=A tryhackme.com` |
| DNS MX record at specific server | `nslookup -type=MX tryhackme.com 1.1.1.1` |
| DNS TXT record | `nslookup -type=TXT tryhackme.com` |
| DNS A record via dig | `dig tryhackme.com A` |
| DNS MX record via dig | `dig @1.1.1.1 tryhackme.com MX` |
| DNS TXT record via dig | `dig tryhackme.com TXT` |

---

## 🧠 Key Lessons

- 🕶️ **Passive ≠ powerless** — WHOIS, DNS, DNSDumpster, and Shodan together can build a surprisingly complete picture of a target with zero direct contact.
- 🧩 **Layer your tools** — WHOIS gives ownership, DNS gives infrastructure, DNSDumpster fills subdomain gaps, and Shodan reveals what's actually exposed live.
- 🕵️ **Forgotten assets are gold** — an obscure subdomain or an old mail server is often where the real weaknesses hide.
- ⚖️ **Passive recon stays legally safer** — it relies purely on public records, unlike active engagement which requires proper authorization.

---

## 🛠️ Tools & Concepts Covered

`whois` `nslookup` `dig` `DNS Record Types (A/AAAA/CNAME/MX/SOA/TXT)` `DNSDumpster` `Shodan.io` `OSINT`

---

## ✅ Status: Completed

*Room completed independently on TryHackMe. Writeup reflects my own notes and understanding of the material — specific flag values are intentionally omitted.*
